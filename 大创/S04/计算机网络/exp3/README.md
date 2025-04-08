---
本科课程: 计算机网络
---
# 实验目标
本次实验在[实验二](Notes/大创/S04/计算机网络/exp2/README.md) 的基础上，完成：
- 实现可靠传输机制
- 在有丢包的网络拓扑上实现文件传输

# 实验前的准备
## 代码文件准备
1. 需要添加仓库中的新文件：
```
exp3/ 
├─ bulk.py # 实现了文件传输功能的python脚本 
├─ tcp_topo_loss.py # 有丢包的mininet测试拓扑
├─ test.py # 测试脚本 
├─ pyarmor_runtime_000000 # 测试脚本依赖文件 
├─ ... 
├─ main.c # main.c 等实验二的文件
```
2. 
- 在实验目录和scripts子目录下执行`chmod +x *.sh`。为所有.sh脚本赋予执行权限
- `./create_randfile.sh` 执行脚本，生成本次实验中用于传输的文件 `client-input.dat`
- 如果报错，可以看 exp2 中的解决方案。

```bash 
sed -i -e 's/\r$//' yourscriptname.sh
```

# 可靠传输
为了实现可靠传输，我们需要：
- 锁
- 滑动窗口

## 锁
为了保证并发安全性，需要进行上锁。越细粒度的锁性能高，但写起来也更容易出错。这里采用三个粗粒度的锁保护 `struct tcp_sock`，框架中没有，需要手动创建在结构体中。
### 需要的锁

- `pthread_mutex_t sk_lock;` 保护`snd_una`等核心参数
    - 收包时：在`tcp_process`调用之前上锁，之后解锁。这样整个收包过程对socket参数的更改都是安全的。
    - 发包时：在`tcp_send_packet`调用之前上锁，之后解锁。这里由于每个人实现不一样，加锁位置不一定一样，比如在调用`` `tcp_send_packet ``之前，用`tcp_tx_window_test`检查发送窗口，那么应该在`tcp_tx_window_test`之前上锁。
- `pthread_mutex_t rcv_buf_lock` 保护`struct ring_buffer *rcv_buf`
    - 每次访问`rcv_buf`时
- `pthread_mutex_t send_buf_lock;`保护`struct list_head send_buf;`
    - 每次访问 `send_buf` 时

**代码修改：** 我在实验二中已经做了一些锁，可以把他们写进结构体，这样会更方便一点，调用起来也更直接明白。
对锁的初始化放在 `alloc_tcp_sock` 中
### sleep_on 和锁w

上锁还有另一个需要注意的问题：**在调用`sleep_on`之前，一定要释放占有的锁**。举一个发送报文的例子：

`pthread_mutex_lock(&tsk->sk_lock); while (!tcp_tx_window_test(tsk)) {     // 如果发送窗口不足，先释放锁，再等待wait_send信号激活     pthread_mutex_unlock(&tsk->sk_lock);     sleep_on(tsk->wait_send);     pthread_mutex_lock(&tsk->sk_lock); } tcp_send_packet(tsk, data_pkt, data_pkt_len); pthread_mutex_unlock(&tsk->sk_lock);`

## 滑动窗口
### 滑动窗口机制：
![滑动窗口图](https://netdocs.lab427.top/images/lab3/Snipaste_2025-03-19_22-19-46.png)

### 怎样分割窗口？
```C
Send Sequence Space

                 1         2          3          4
            ----------|----------|----------|----------
                   SND.UNA    SND.NXT    SND.UNA
                                        +SND.WND

      1 - old sequence numbers which have been acknowledged
      2 - sequence numbers of unacknowledged data
      3 - sequence numbers allowed for new data transmission
      4 - future sequence numbers which are not yet allowed

                        Send Sequence Space
```
`snd_una, snd.nxt, snd.una, snd.wnd` 都以字节为单位. （看看 wireshark 的 len 和 seq ack 增长的关系就理解了）

### 发送方(Sender)行为：
发送方通过计算某次发送包所处在的窗口来判断自己行为
- 已经被接收的字节：`seq < snd_una`
- 发送了，但为收到ACK的字节：`snd_una <= seq < snd_nxt`
- 未发送，且当前发送窗口剩余大小允许继续发送的字节：`snd_nxt <= seq < snd_una + snd_wnd`
- 发送窗口剩余大小不允许继续发送的字节：`seq >= snd_una + snd_wnd`
### 接收方(Receiver)行为
**接收方：**

接收方采用**累计确认机制**进行回复，按照课本的GBN机制，接收方是不维护窗口的。不过一般还是会维护乱序队列（Out-of-Order Queue），对应于`tcp_sock`中的`rcv_ofo_buf`，这是下一节要完成的内容，本节还是假设没有窗口。

累计确认机制下，
- 接收端发送的ACK报文的应答序号 `tcp->ack` 始终是自己希望收到的下一个字节 `tsk->rcv_nxt`
- 这一点与是否实现乱序队列无关，因而实验中发送ACK报文只需要调用 `tcp_send_control_packet(tsk, ACK)`，序号的设置会由函数自动进行。

在本节中，接收方收到一个报文，需要检查该报文的序号（范围 `[cb->seq, cb->seq_end)`）和自己期望收到的序号 `tsk->rcv_nxt` 的关系。
注意，比较序列号时最好使用 `tcp.h` 中提供的比较函数 `less_than_32b` 等，它们会自动处理整数的**溢出问题。**（此溢出问题在 exp2 中遇到过）
- `cb->seq_end <= tsk->rcv_nxt`：收到了之前确认过的报文，不需要处理报文中的数据，直接回复ACK。这种情况在无丢包时不应该出现，但可以添加这一处理，防止python侧发送这样的报文（例如KeepAlive报文）。
- `tsk->rcv_nxt == cb->seq`：刚好是自己希望收到的报文，将数据上送`rcv_buf`
- `tsk->rcv_nxt < cb->seq`：乱序报文，后面处理，这里直接回复ACK，数据丢弃。
- 其它情况不应出现