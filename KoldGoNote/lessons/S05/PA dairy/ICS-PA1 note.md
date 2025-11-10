# 尝试编译NEMU

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250829140825.png)

成功进入 Configuration 界面。

根据 `PA0` 讲义的内容，成功操作 `make` 并 `run` 了 `NEMU`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250829144056.png)

根据讲义的内容：该断言是正常的，我们将在 PA1 解决该问题；

> This message tells you that the program has triggered an assertion fail at line 36 of the file `nemu/src/monitor/monitor.c`. If you do not know what is assertion, blame the 程序设计基础 course. But just ignore it now, and you will fix it in PA1.
   To debug NEMU with gdb, type

```
make gdb
```

## 关于用 ssh 传输文件的问题
该问题的需求是：我希望能将服务器文件保存一些到本地，同时推送我在 Mac 上下载好的红白机游戏文件到服务器，这就需要文件传输

**gemini 老师如此教学：**

---

SFTP (SSH File Transfer Protocol) 也是一种基于 SSH 的文件传输协议，但它与 `scp` 有所不同。`scp` 更像一个简单的复制命令，而 SFTP 则提供了一个**交互式的环境**，让你可以在服务器上进行类似文件管理器（如 Windows 文件资源管理器）的操作。

你可以把 SFTP 想象成一个功能更强大的 FTP，但它通过 SSH 进行加密传输，所以更安全。

-----

### 如何使用 SFTP 推送文件？

使用 SFTP 的基本流程是先建立连接，然后在连接中执行文件操作。

#### 1\. 建立 SFTP 连接

在终端输入以下命令来连接你的服务器：

```bash
sftp username@server_ip
```

  * **`username`**：你的服务器用户名。
  * **`server_ip`**：你的服务器 IP 地址或域名。

如果你使用了非默认的 SSH 端口，可以用 `-P` 选项来指定：

```bash
sftp -P 12345 username@server_ip
```

连接成功后，你会进入一个交互式 SFTP 命令行界面，提示符通常是 `sftp>`。

#### 2\. 在 SFTP 界面中推送文件

进入 SFTP 界面后，你可以使用各种命令来管理文件。

  * **`lpwd`**：查看你在**本地**的当前路径。
  * **`pwd`**：查看你在**服务器**上的当前路径。
  * **`lls`**：列出**本地**当前目录下的文件。
  * **`ls`**：列出**服务器**当前目录下的文件。
  * **`lcd`**：切换**本地**目录。例如：`lcd /Users/myname/Desktop`。
  * **`cd`**：切换**服务器**目录。例如：`cd /var/www/html`。
  * **`put`**：上传文件。例如：`put my_local_file.txt` 将本地文件上传到服务器当前目录。
  * **`get`**：下载文件。例如：`get my_remote_file.txt` 将服务器文件下载到本地当前目录。

**要推送文件，只需使用 `put` 命令**：

```bash
sftp> put /path/to/my_local_file.txt
```

如果只写文件名，`sftp` 会在本地当前目录寻找这个文件。

#### 3\. 退出 SFTP

当你完成文件传输后，输入 `exit` 或 `bye` 即可退出 SFTP 界面。

```bash
sftp> exit
```

-----



# 检查 xfce4+xrdp 是否可以正常运行游戏

![](https://kold.oss-cn-shanghai.aliyuncs.com/20250901150811.png)

## 阅读代码
### 内存管理

> x86的物理内存是从0开始编址的, 但对于一些ISA来说却不是这样, 例如mips32和riscv32的物理地址均从 `0x80000000` 开始. 因此对于mips32和riscv32, 其 `CONFIG_MBASE` 将会被定义成 `0x80000000`. 将来CPU访问内存时, 我们会将CPU将要访问的内存地址映射到 `pmem` 中的相应偏移位置, 这是通过 `nemu/src/memory/paddr.c` 中的 `guest_to_host()` 函数实现的.

`guest_to_host` 的实现比较简单

```c
uint8_t* guest_to_host(paddr_t paddr) { return pmem + paddr - CONFIG_MBASE; }
```


```c
// nemu/src/nemu-main.c
   int main(int argc, char *argv[]) {
    /* Initialize the monitor. */
   #ifdef CONFIG_TARGET_AM
     am_init_monitor();
   #else
     init_monitor(argc, argv);
   #endif 
   
       /* Start engine. */
     engine_start();                                                                                  
     
     // a test function
   
⚠    printf("test before engine start\n");
   
     return is_exit_status_bad();
   }
```

---
下面是一个小问题的回答。

> [!Note] 究竟要执行多久?
> 在 `cmd_c()` 函数中, 调用 `cpu_exec()` 的时候传入了参数 `-1`, 你知道这是什么意思吗?

- 我们查看 `cpu_exec()` 源码

```c
//cpu_exec()
/* Simulate how the CPU works. */
void cpu_exec(uint64_t n) {
  g_print_step = (n < MAX_INST_TO_PRINT);
  switch (nemu_state.state) {
    case NEMU_END: case NEMU_ABORT: case NEMU_QUIT:
      printf("Program execution has ended. To restart the program, exit NEMU and run again.\n");
      return;
    default: nemu_state.state = NEMU_RUNNING;
  }
//...
}
```


可见传入参数 `-1` 代表无限循环，因为 `-1` 总是小于。类似不加限制的开始运行。

### 我需要做哪些事情

#### 完善 monitor（自实现简单版 gdb）

在熟悉框架代码的基础上，我们来看

| 命令      | 格式            | 使用举例                   | 说明                                                                                                              |
| ------- | ------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------- |
| 帮助(1)   | `help`        | `help`                 | 打印命令的帮助信息                                                                                                       |
| 继续运行(1) | `c`           | `c`                    | 继续运行被暂停的程序                                                                                                      |
| 退出(1)   | `q`           | `q`                    | 退出NEMU                                                                                                          |
| 单步执行    | `si [N]`      | `si 10`                | 让程序单步执行`N`条指令后暂停执行,  <br>当`N`没有给出时, 缺省为`1`                                                                      |
| 打印程序状态  | `info SUBCMD` | `info r`  <br>`info w` | 打印寄存器状态  <br>打印监视点信息                                                                                            |
| 扫描内存(2) | `x N EXPR`    | `x 10 $esp`            | 求出表达式`EXPR`的值, 将结果作为起始内存  <br>地址, 以十六进制形式输出连续的`N`个4字节                                                           |
| 表达式求值   | `p EXPR`      | `p $eax + 1`           | 求出表达式`EXPR`的值, `EXPR`支持的  <br>运算请见[调试中的表达式求值](https://nju-projectn.github.io/ics-pa-gitbook/ics2024/1.6.html)小节 |
| 设置监视点   | `w EXPR`      | `w *0x2000`            | 当表达式`EXPR`的值发生变化时, 暂停程序执行                                                                                       |
| 删除监视点   | `d N`         | `d 2`                  | 删除序号为`N`的监视点                                                                                                    |


### 需要注意的小 TIPs

#### 常用的宏文件
最后我们聊聊代码中一些值得注意的地方.>
- 三个对调试有用的宏(在 `nemu/include/debug.h` 中定义)
    
    - `Log()`是`printf()`的升级版, 专门用来输出调试信息, 同时还会输出使用`Log()`所在的源文件, 行号和函数. 当输出的调试信息过多的时候, 可以很方便地定位到代码中的相关位置
    - `Assert()`是`assert()`的升级版, 当测试条件为假时, 在assertion fail之前可以输出一些信息
    - `panic()`用于输出信息并结束程序, 相当于无条件的assertion fail
    
    代码中已经给出了使用这三个宏的例子, 如果你不知道如何使用它们, RTFSC.

## Stage-1 实现简易调试器 SDB

### 解决按 `q` 退出报错问题

在 `src/utils/state.c` 中，定义了 `nemu_state` 作为全局变量
`NEMUState nemu_state = { .state = NEMU_STOP };`
我们按 `q` 退出 `nemu` 的时候，会报错

```c
#include <common.h>

void init_monitor(int, char *[]);
void am_init_monitor();
int is_exit_status_bad();
void engine_start();

int main(int argc, char *argv[]) {
  /* Initialize the monitor. */
#ifdef CONFIG_TARGET_AM
  am_init_monitor();
#else
  init_monitor(argc, argv);
#endif 

    /* Start engine. */
  engine_start();
  
  // a test function

  printf("test before engine start\n");

  return is_exit_status_bad();
}
```

通过加一行 printf，发现是 `is_exit_status_bad` 的问题。

```c
int is_exit_status_bad() {
	int good = (nemu_state.state == NEMU_END && nemu_state.halt_ret == 0) ||
	(nemu_state.state == NEMU_QUIT);
	   return !good;
}           
```

这个函数，会检查是否 `nemu_state.state` 是 `good` 的。可见问题在于我们没有在按下 `q` 事件后，更新状态
### 初览 `sdb.c`

SDB 代码位置在
`nemu/src/monitor/sdb` 处

```c
// sdb.c
static struct {
  const char *name;
  const char *description;
  int (*handler) (char *);
} cmd_table [] = {
  { "help", "Display information about all supported commands", cmd_help },
  { "c", "Continue the execution of the program", cmd_c },
  { "q", "Exit NEMU", cmd_q },

  /* TODO: Add more commands */

};

```

这个结构体 `cmd_table` 存储了一些命令的介绍。我们后面添加命令，也从这里坐。

`handler` 是用来定位这个命令的

`cmd_table` 中已经有了三个实现好的指令


### 实现按步执行
```c
static int cmd_si(char* args)
{
    char* arg = strtok(args, " ");

    int steps = 0;

    if (arg == NULL)
    {
        // if not find N arguments
        steps = 1;
    }
    else
    {
        // An argument is provided, Convert it to an integer
        steps = atoi(arg);
        if (steps <= 0)
        {
            Log("Invalid argument!");
            return -1;
        }
    }

    cpu_exec(steps);

    return 0;
}
```
对于 `steps<=0` 的处理，我个人觉得将其作为未设定值更合适。因为持续运行到结束，是 `cmd_c` 的工作

### 表达式求值

相关文件：`src/monitor/sdb/expr.c`
#### 语法分析

用到的工具： **正则表达式**
框架代码提供了一系列规则，需要我们继续添加。

这里需要注意 C 语言的转义字符问题。`+` 、`*`、`()` 都是正则表达式自己用的 **符号**，我们需要在前面加上转义符号 ` \ `，但是 ` \` 本身也是特殊符号，所以是 `\\+`
```c
// src/monitor/sdb/expr.c

// token类型枚举
enum {
  TK_NOTYPE = 256, TK_EQ,

  /* TODO: Add more token types */

};

// 正则表达式规则
static struct rule {
  const char *regex;
  int token_type;
} rules[] = {

  /* TODO: Add more rules.
   * Pay attention to the precedence level of different rules.
   */

  {" +", TK_NOTYPE},    // spaces
  {"\\+", '+'},         // plus
  {"==", TK_EQ},        // equal
};
```
现在需要分析的：
- 十进制整数
- `+`, `-`, `*`, `/`
- `(`, `)`
- 空格串(一个或多个空格)

之后，用 `make_token` 函数，按照规则处理，将分析出的 token 存在 `tokens` 数组


### 实现监视点

- 模拟内存位置
- `nemu/src/memory/paddr.c`
	- host 客户程序，存在模拟内存中
- 内存通过在 `nemu/src/memory/paddr.c` 中定义的大数组作为内存
#### 拓展表达式求值


#### 监视点池

> 你从大一到大三成长了两年多，**仍然要对最初困扰你的链表抱有敬畏**



