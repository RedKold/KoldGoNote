
# 实验 3 同步时序部件设计


## 1. 计数器实验 
### 实验内容：
根据表3.1 给出的功能表和图 3.1 所示电路原理图构建 4 位二进制同步计数器电路。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027190539.png)


### 基本原理
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027190554.png)


### 设计电路
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027192542.png)


### 仿真测试
验证电路功能无误。

### 修改封装图。


## 2.  移位寄存器实验

### 实验内容
根据表3.2 给出的功能描述和图 3.5 给出的电路原理图，构建 4 位通用移位寄存器电路SHRG4U 该移位寄存器带有异步复位（清 0）信号CLR，它是低电平有效信号，当它为低电平时，所有 D 触发器的状态输出为 0。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027202816.png)
- 功能表
### 基本原理
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027202802.png)
- 原理图如图所示

### 设计电路
电路图如图所示

### 仿真测试

经测试，运行正常。![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027202901.png)
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027202936.png)



### 错误记录

![233.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/233.png)

一开始，忘记了连接 `CLR`，导致 `CLR` 功能在低电平时候没有清空位数。


## 3  4位无符号数乘法器
### 实验内容
实验将实现两个四位二进制无符号数相乘的功能并通过数码管将其转换成十六进制显示出来。

### 基本原理
**类似于手算乘法**。
区别只在：
在每次求得 X×Yi 后，不是将它左移与前次部分积 Pi 相加，而是将部分积 Pi 右移
一位与 X × Yi 相加。
为乘数为 `0` 的 bit 可以直接执行右移。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105125138.png)

### 基本电路

![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251029194230.png)



### 错误记录
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251029193545.png)


错误连接了寄存器的使能端（应该接 `clk`）和 `rst`），导致没有完成寄存工作。



## 4 寄存器堆实验

### 实验内容
中寄存器堆原理图，构建含有 **32 个 32 位寄存器的寄存器堆 Regfile** 的读写电路，包含两个读数据端口和一个写数据端口，并封装成子电路。寄存器堆的读操作属于组合逻辑操作，无须时钟控制，即当寄存器地址信号 RA 或 RB 到达后，经过一个 “读取时间 ”的延迟，读出的数据输出到端口busA 或 busB 上。寄存器堆的写操作则属于时序逻辑操作，需要时钟信号的控制，即在写使能信号（ WE）有效的情况下，有效时钟触发边沿到来后将写入数据端口 busW 上的信息写入 RW 所指定的寄存器中。

### 基本原理

设计原理图如图所示
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105125333.png)

简单来说就是通过 `32-1 decoder` 找到要操作的寄存器，然后再通过 MUX 来写入或读取数据并送到 bus。

### 基本电路
- 局部示意图

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251029202312.png)

![lab3-3.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/lab3-3.png)

### 仿真测试

经测试，可以正常完整任务

- `read test`
	- RA 读取 3 号寄存器，RB 读取 2 号寄存器。一切正常
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105130341.png)


- `write test`
- 写入 1 号寄存器。其按 `16` 进制正确显示了
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105130235.png)

### 错误记录

该实验过程未遇到错误。
> [!Warning] **注意**
> 但是要格外注意，这个实现 32 个寄存器，连线众多，不要看走眼



## 5 时钟

### 实验内容

> [!Note] **实验要求**
> 在6 个 7 段数码管上显示数字时钟时分秒，当计时到 23:59:59后进入 00:00:00，时分秒之间用小数点分隔；
> 整点时点亮三色 LED 灯组件，按照格雷码赋值，每秒显示一种颜色，持续显示 10 秒
> 通过8421BCD 码设置初始时间，在载入时，如果初始时间数值超出实际范围，则报输入错，且不能被载入。

### **基本原理**

本实验的时分秒可以靠**与门**来实现，同时要注意：高位要发生进位，一定是低位都传来进位信号的情况下，且本位也要进位，**才能进位**。


### 基本电路
本电路元素众多，我们拆解来看

- **计时单元模块**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105131142.png)
- `load` 错误检测模块
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105131218.png)

- `led` 模块
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105131238.png)
	- **这里实验文档有点模糊**。实际是 8 个一循环的格雷码，只需要用三位。
	- 格雷码的知识请回顾[[DLCO-BooleanAlgebra#格雷码|格雷码知识]]
		`grey(a) == a ^ (a>>1)`
### 仿真测试
通过了 OJ **所有测试样例**，未发现问题。
- `load` 测试
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105131617.png)
- `load` 的 `err` 检测
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105131638.png)
- 进位处理
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105131718.png)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105132140.png)

- 亮灯测试：
	- 实际上述进位已经触发了亮灯测试。OJ 对亮灯进行了完整测试，**也无问题**。
### 错误记录

![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20251029233714.png)
这里 OJ 时候出错了，发现是 `LD` load 信号为 `0` 的时候，错误的反悔了 `INERR`, 而实际实验要求不 `load` 的时候，不对此做检测。
**加一个与门检测一下即可**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251105131112.png)

## 思考题

1. 修改同步计数器电路设计，实现异步清零和异步置数的 4位计数器，并说明该计数器的应用特性。
2. 如何仅利用 4位 同步加法 计数器 在 不 增 加其它逻辑门的情况下，实现模 10计数器。
3. 利用 4位移位寄存器 设计 8位二进制伪随机 序列 电路 ，写出输出序列值 。
4. 如何 实现 寄存器堆 的写后读 功能 。


### 修改同步计数器电路设计，实现异步清零和异步置数的 4位计数器，并说明该计数器的应用特性。
**异步**即不依赖时钟信号，可以立刻完成操作。

而 `D` 触发器已经提供了异步的接口，上部接入 1 会异步 `set 1`，下部接入 `0` 会异步 `set 0`，我们利用这一点设计：
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251106125948.png)

即可完成。
电路说明：`AS-CLR` 是异步置零信号，`AS-LD` 是异步 `load` 信号，`AS-D` 是异步 set 的 `4bit` value
我们用与门来完成：`when AS-LD, load AS-D`, 高电平则接入对应触发器的异步 `0`，低电平则接入对应触发器的异步 `1`。
同时 `set 0` 由于要和 `AS-CLR` 兼容，用一个或门。

仿真测试能完成任务。

### 如何仅利用 4 位同步加法计数器在不增加其它逻辑门的情况下，实现模 10 计数器。

模 `10` 意味着什么？在到达 `10` 之后 set `0` 继续工作。
这和时钟很像。时钟的分秒实际就是一个 `模60` 系统。

我们使用比较器：结果 `>10` 即载入 0

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251107141958.png)

### 利用 4 位移位寄存器设计 8 位二进制伪随机序列电路，
> [!Note] 什么是 **伪随机数列**？
> 一种由确定性电路或算法产生、看起来“像随机”的数列，并非真正随机。

可以用 `LFSR` 线性反馈移位寄存器来做。(Linear Feedback Shift Register)
- **反馈函数**(feedback function) 决定了序列规律。
- 我们可以用 $P(x)=x^{4}+x+1$ 来做这个多项式，则 **反馈函数** 为 $f=Q_{3} \oplus Q_{0}$
- 所以我们只需要每次移位对 `Q3` 和 `Q0` 这两个寄存器做异或即可输出这个序列。而最初移位的对象，**就是我们随机数的种子**。

- 假设初始状态为 `0001`

|时钟周期|当前状态 Q₃Q₂Q₁Q₀|反馈 f=Q₃⊕Q₀|下一状态|输出（Q₀）|
|---|---|---|---|---|
|1|0001|1|1000|1|
|2|1000|1|1100|0|
|3|1100|0|0110|0|
|4|0110|0|0011|0|
|5|0011|1|1001|1|
|6|1001|0|0100|1|
|7|0100|0|0010|0|
|8|0010|0|0001|0|
|9|0001|1|1000|1|

### 如何实现寄存器堆的写后读功能。
clarify 一下：**寄存器堆**的写后读指的是同一时钟周期，写入并读取后，读取到的是新值。

为了不破坏时序结构引入异步，我们可以引入“写入地址”和“读取地址”的比较。

**引入比较器**：
- 如果 `write address == read address`
	- `output write value`
- else
	- `output read value`
- **具体实现可以引入一个多路选择器**。
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251107143721.png)
