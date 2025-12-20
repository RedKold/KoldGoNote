 
# 实验 6 单周期 CPU 设计与测试




## 1. 控制器设计实验
### 实验内容：

实现一个 RV32I 指令集的控制器
我们要实现的控制器如下：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219223024.png)


### 基本原理

可以根据不同指令的 `op` `func3` `func7` 等唯一确定控制信号的值，然后根据最小风险法化简即可。实验手册中已经给出了详尽的控制信号真值表。

我们发现，相同类型指令的操作码基本相同，所以可以分为几大类标志位来简化处理。


### 设计电路

设计电路图如下

- 产生指令分类的标志位
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219223224.png)

- 得到 `RegWr`, `MemtoReg`, `MemWr` 等信号：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219223309.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219223325.png)
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219223334.png)

### 仿真测试

通过了所有 OJ 测试样例，且后面依赖未出现问题，验收成功，这里省略额外的仿真测试。

## 2. 单周期 CPU 设计实验

### 实验内容
实现单周期 CPU 的设计。

### 基本原理
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219223939.png)

单周期 CPU 的状态单元
- PC 寄存器
- 指令存储器
- 数据存储器
我们将之前实验设计的不同部件组装在一起即可完成任务



### 设计电路

增加一个 `halt` 端口：

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219224122.png)

这样如果执行 `ecall` 并设置 `halt=1`，PC 寄存器就不会再更新了

### 仿真测试

通过了所有 OJ 测试样例，且后面依赖未出现问题，验收成功，这里省略额外的仿真测试。

## 3 累加和程序测试实验
### 基本原理

#### Test in RARS
我们在 RARS 的代码编辑中用 asm 写一个累加和程序，然后编译成 RV32 的指令序列
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251217132806.png)

编译成功后，在 `Data Segment` 的 `0x4` 放 `0x64`，观察结果

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251217133040.png)

可以看到结果为 `0x000013ba`，且 `success` 寄存器 `a0` 的内容为 `0x00c0ffee`，说明执行成功

分别输入参数 `0x0f, 0x0ff, 0x0f00, 0x0ff00` 来测试：
- `0x0f`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251217133255.png)
- 累加到 16，结果为 `0x78` 即 `120`，符合题意。$\sum_{i=1}^{15}i=120$
- `0x0ff`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251217133543.png)
 - `res = 0x7f80`, success
- `0x0f00`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251217133612.png)
- `res = 0x708780` success
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251217133710.png)

- `res = 0x7f00ff80` success


#### Test in Logisim
我们首先将 `lab6.2.circ` 的电路装入 `lab6.3`，然后在指令存储器装入代码文件（从 RARS 中可以导出）
- 为了和从 OJ 平台的区分开来，存储为 `lab6.3-myown.hex`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251217133913.png)


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251213212153.png)

修改 `DataRAM` 中的最低位 `RAM0` 的值，作为 `n=100`

进行仿真测试，发现电路最后停止在 `halt` 指令
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251213213725.png)


查看 `n=100` 时存储器的结果：
![image.png|900](https://kold.oss-cn-shanghai.aliyuncs.com/20251213213716.png)
符合实验报告中理想结果



### 基本电路
电路图和单周期 CPU 设计实验完全一致

## 4 冒泡排序程序测试实验

和实验 3 类似，无需对电路做修改。已通过 OJ 测试。
对于冒泡程序执行的正确性，可以参考 [[#5 官方测试集实验]]

## 5 官方测试集实验
### 基本原理
官方测试集针对不同的RISC-V 指令变种都提供了测试。在本实验中主要使用 rv32ui 也就是 RV32 的基本指令集， u 表示是用户态， i 表示是整数基本指令集。实验中采用的环境是无虚拟地址的环境，即只使用物理地址访问内存。所以，主要关注 rv32ui-p 开头的测试即可。

### 测试结果

详情见附表记录。
已和助教完成抽查。
## 6 ICS PA 程序测试实验

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251219222648.png)
我添加了这样的小电路来方便读取 RAM 中的信息（因为我实际通过 4 个 int8 的 RAM 来模拟）。

在执行 PA 的某指令 `0x6f` 发现会无限循环，反汇编发现 `0x6f` 是
```asm
jal zero, 0
```

即跳转到自己。

若让时钟可以停下，可以加一个判断，也归到 `0x73`` ecall` 指令即可。

- **补充**：我发现实验手册其实描述了这个现象
> 提示：在测试文件机器代码中可能使用“0000006f”来结束程序，需要修改为 00000073”；或者增加生成中止信号的逻辑条件，当指令为“ 0000006f”时，中止信号 halt也有效，赋值为 1。

This ends the DLCO lab6.

## 思考题
### 如何在单 CPU 上实现多任务处理，例如同时执行计算累加和与数据排序两个程序，阐述思路。

单 CPU 实现多任务处理，实际是一个**并发**的问题。
让累加和和数据排序 **轮询**这一个 CPU 来执行。即累加和先执行一条或若干条指令，然后等待数据排序执行一条或若干条指令，交替进行即可。

但是为了程序执行的正确性，我们还需要**保存现场**

当从一个任务切换到另一个时，必须保存当前任务的状态，恢复下一个任务的状态。
- 通用寄存器内容（如 RAX, RBX, ...)
- 程序计数器（PC / EIP）
- 栈指针（SP）、状态寄存器（FLAGS）
- 内存映射信息

- 同时，为了避免程序互相冲突，**我们应该把程序放在不同的内存位置**
### 在 CPU的基础上，如何实现键盘输入、 TTY输出部件等输入输出设备的数据访问，构建完整的计算机系统。

~~ICS PA is All You Need~~

我们可以把键盘输入，TTY 输出部件等都抽象成文件，为他们规定不同的 `I/O` 方式。

**一个可行的方案是内存映射**，将数据存在内存中的特定位置，CPU 读取这个内存来获得键盘、显存 TTY 信息


```
+---------------------+
|     User Program    |   ← open(), read(), write(), ioctl()
+---------------------+
|   VFS (虚拟文件系统) |   ← 提供统一接口，“一切皆文件”
+---------------------+
|   字符设备驱动       |   ← keyboard.c, serial.c, console.c
+---------------------+
|   硬件抽象层 (HAL)   |   ← MMIO/PMIO 访问，中断注册
+---------------------+
|     Hardware        |   ← Keyboard, UART, VGA Controller
```

笔者事实上在 DLCO 学期已经选修了 ICS，做了 PA，可以阅读博客 [ICS-PA日记-PA2 | RedKold的小站](https://redkold.github.io/2025/11/06/ics-pa2/)
### 阐述如何在单周期 CPU 基础上实现多周期 CPU
通过把每条指令分为**不同阶段**，每个阶段使用一个独立的时钟周期，这样就把每条指令划分为了多个时钟周期。

阶段划分要注意原子化，完成一个独立的小功能。

- 每个时钟内的执行结果在下个时钟到来前，保存到相应存储元件或者稳定地保持在组合电路中
- **时钟周期的宽度以最复杂**阶段的所用的时间为准。
- 不同指令有不同的周期数，但我们以最长的为准设计。

然后，我们设计不同阶段的跳转逻辑（根据不同指令而定），形成一个状态机，然后，就是按照时序逻辑的方法设计电路的过程了。

### 阐述如何在单周期 CPU 基础上实现流水线 CPU
**流水线**要实现一个专门的流水线控制器，在 `ID(Instruction Decode)` 阶段就要生成所有控制信号
同时还要维护一些流水线段寄存器，保存执行中信息。

一个指令执行分为不同阶段：
拿 Load 举个例子：
```
Load：
+-------------------+
|	IF(Inst Fetch)	|
+-------------------+
|	Reg/Dec			|  # When Load at this stage, another inst start to step onto IF
+-------------------+
|	EX				|
+-------------------+
|	M				|
+-------------------+
|	WB				|
+-------------------+
```

后续指令，将是在 Load 执行 Reg/Dec 阶段开始自己的第一阶段。**像流水线一样周而复始**。


值得注意的是这种方法，不需要有限状态机。(Finite State Machine)

