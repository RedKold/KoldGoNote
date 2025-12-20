> 姓名：朱晗
> 学号：`231275036`



# 实验 4 运算部件设计


## 1. 4 位先行进位加法器 CLA
### 实验内容：
实现 4 位先行进位加法器 CLA

我们需要先实现先行进位部件 (Carry Lookahead Unit, CLU)，**结合全加器**就可以完成任务。
### 基本原理
```
C1 = G0+ P0C0
C2 = G1 + P1G0 + P1P0C0
C3 = G2 + P2G1 + P2P1G0 + P2P1P0C0
C4 = G3 + P3G2 + P3P2G1 + P3P2P1G0 + P3P2P1P0C0
```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210516.png)

同时，**我们为了拓展性**，将组间进位生成函数 `Gg` 和传递函数 `Pg` 也抽象出来 

```
Pg= P3P2P1P0
Gg= G3+P3G2+P3P2G1+P3P2P1G0
```
### 设计电路

- CLU 4 如图所示
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210722.png)
- 设计全加器则如图所示
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210648.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210804.png)
封装如图


### 仿真测试
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210828.png)
- 加法测试

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210852.png)
- **进位测试**


## 2. 16 位两级先行进位加法器实验
### 基本原理
对于一个 16 位加法器，可以分成 4 组，每组用一个 4 位先行进位加法器 CLA 实现

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210922.png)

### 设计电路

电路设计如图所示
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201210952.png)


### 仿真测试
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201211036.png)
- 加法测试
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201211053.png)
- **进位测试**


## 3 32 位快速加法器实验
### 基本原理
通过将两个 16 位两级先行进位加法器串行级联构建一个 32 位加法器，并根据给出的标志位生成电路原理图，在 32 位加法器中生成 CF、SF、OF、ZF 标志位。

原理是一样的。

标志位有一些新的需要注意的：
$OF=\overline{A_{n-1}}\cdot\overline{B_{n-1}}\cdot F_{n-1}+A_{n-1}\cdot B_{n-1}\cdot\overline{F_{n-1}}$
$SF=F_{n-1}$
$CF=Cout\otimes Cin$
$ZF=\overline{F_{n-1}+F_{n-2}+\cdots+F_{0}}$




### 基本电路

我们 $ZF$ 直接采取和 `0` 比较器的方式获得。
当然，组合逻辑分组获取也可以，效率可能更高

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201211632.png)
- 该电路图的标志位和运算主逻辑分开，显示更清晰一些

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201211742.png)
- **封装图如图所示**

### 仿真测试
通过了所有测试样例。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201211913.png)
- 标志位生成
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201211933.png)
- 零标志位测试




## 4 32 位桶形移位器实验
这是之前的思考题，具体实现类似，只是规模更大。

### 基本原理
同 4 位桶形移位器。

### 电路设计
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201212012.png)

整体设计如图所示。

具体来说，上方设计左移，下方设计右移
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201212121.png)
通过 MUX **实现不同的位移方式**。

通过每次将位移结果送下一个位移单元，我们可以通过 `16 8 4 2 1` 的四组位移单元凑出 `0-15` 的移位可能，满足 32 位桶形移位器的要求

## 5 RV32I 算术逻辑部件实验
算术逻辑部件 ALU 是 CPU 中的核心数据通路部件之一，它主要完成 CPU 中需要进行的算术逻辑运算。
RV32I 包括 4 种 ALU 指令：比较运算、移位运算、逻辑运算和算术运算。
比较运算通过操作数相减然后判断标志位来生成结果；移位运算通过桶形移位器来实现左移和右移；
逻辑运算可直接通过逻辑门阵列来实现；
算术运算通过加法器以及生成的标志位来实现。
ALU 在操作控制信号 ALUctr 的指示下，执行相关运算，Result 作为 ALU 运算的结果被输出，零标志 Zero 被作为 ALU 的结果标志信息输出。
### 基本原理

我们首先用一个子电路构造 `Decoder`，也就是简化的 `Decode` 指令过程，具体来说从指令取出控制信号和 `OPctr`

文档已经给出了 `OPctr` 和不同 `control-signal` 的最小项表达式
```
SUBctr =Σm(2,3,8)
SIGctr = m2
ALctr = m13
LRctr = m1
Opctr[2]= Σm(1,2,3,5,13,15)
Opctr[1]= Σm(2,3,4,6)
Opctr[0]= Σm(4,7,15)
```
我们可以很方便的实现。

实现之后，将 A 和 B 结果进行各种运算，接一个 MUX，按照 `OPctr` 的值选择就好了。


### 电路设计

**首先设计 `ALUctr`**
我们先从 `ALUctr` 信号取出各个子项，**然后根据最小项式子构造就可以了**。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201212925.png)


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201213006.png)

- 我们依次按照各种计算获取值，连到 MUX 上即可。
这里需要注意无符号和有符号的处理：
- 对于有符号减法，我们信号是 `SIGctr=1, SUBctr=1`，这个情况我们要对 B 取其补码作为 `-B`，具体而言对 `ffff ffff` 取异或即可（也就是各位取反），然后末位+1（这在加法器的 `Cin` 信号可以变相实现，很巧妙）
- 还要注意一个无符号小于 0 和有符号小于 0 置 0 的处理。

### 仿真测试
通过了所有 OJ 测试。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201220357.png)
- 移位测试
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201220428.png)
- **减法测试**


### 功能测试


| 序号  | A          | B          | ALUctr<3:0> | Result       | Zero |
| --- | ---------- | ---------- | ----------- | ------------ | ---- |
| 1   | 0x80000000 | 0x7fffffff | 0000        | `0xffffffff` | 0    |
| 2   | 0x01       | 0x7fffffff | 0001        | `0x0`        | 0    |
| 3   | 0x80000000 | 0x7fffffff | 0010        | `0x1`        | 0    |
| 4   | 0x80000000 | 0x7fffffff | 0011        | `0x0`        | 0    |
| 5   | 0x80000000 | 0x7fffffff | 0100        | `0xffffffff` | 0    |
| 6   | 0x80000000 | 0x7fffffff | 0101        | `0x1`        | 0    |
| 7   | 0x80000000 | 0x7fffffff | 0110        | `0xffffffff` | 0    |
| 8   | 0x80000000 | 0x7fffffff | 0111        | `0x0`        | 0    |
| 9   | 0x80000000 | 0x7fffffff | 1000        | `0x1`        | 0    |
| 10  | 0x80000000 | 0x7fffffff | 1101        | `0xffffffff` | 0    |
| 11  | 0x80000000 | 0x7fffffff | 1111        | `0xffffffff` | 0    |
| 12  | 0x80000000 | 0x80000000 | 0000        | `0x0`        | 1    |
| 13  | 0x80000000 | 0x80000000 | 0010        | `0x0`        | 1    |
| 14  | 0x80000000 | 0x80000000 | 0011        | `0x0`        | 1    |
| 15  | 0x80000000 | 0x80000000 | 0100        | `0x0`        | 1    |
| 16  | 0x80000000 | 0x80000000 | 0101        | `0x80000000` | 1    |
| 17  | 0x80000000 | 0x80000000 | 0110        | `0x80000000` | 1    |
| 18  | 0x80000000 | 0x80000000 | 0111        | `0x80000000` | 1    |
| 19  | 0x80000000 | 0x80000000 | 1000        | `0x0`        | 1    |
| 20  | 0x80000000 | 0x80000000 | 1101        | `0x80000000` | 1    |
| 21  | 0x80000000 | 0x80000000 | 1111        | `0x80000000` | 1    |
| 22  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0000        | `0x4b4b4b4a` | 0    |
| 23  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0001        | `0xb4b4b4a0` | 0    |
| 24  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0010        | `0x0`        | 1    |
| 25  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0011        | `0x0`        | 1    |
| 26  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0100        | `0x0`        | 0    |
| 27  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0101        | `0x052d2d2d` | 0    |
| 28  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0110        | `0xa5a5a5a5` | 0    |
| 29  | 0xa5a5a5a5 | 0xa5a5a5a5 | 0111        | `0xa5a5a5a5` | 0    |
| 30  | 0xa5a5a5a5 | 0xa5a5a5a5 | 1000        | `0x0`        | 1    |
| 31  | 0xa5a5a5a5 | 0xa5a5a5a5 | 1101        | `0xfd2d2d2d` | 0    |
| 32  | 0xa5a5a5a5 | 0xa5a5a5a5 | 1111        | `0xa5a5a5a5` | 0    |
|     |            |            |             |              |      |


## 思考题
### 通过查找资料，实现一种 64 位二进制快速加法器的设计。

对于 64 位二进制快速加法器，我们可以通过分别用两个 32 位数存储高位和低位的方式。然后将中间结果传送给给高位作为 Cin 即可。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201222003.png)



### 将实验 3 中的快速乘法器设计电路扩展到 32 位无符号数相乘，并探讨如何将该乘法器融合到实验中的 ALU 电路来实现乘法运算。

扩展到 32 位相乘，我们这时候需要将移位器和加法器等子部件都替换成新的 32 位的
**注意到乘法器** 只需要每次移位 1，**所以我们可以用分线器**实现移位，避免复杂操作。
此题目如果严格来说，需要对 Z 取高位 32bit 和低位 32bit 分别计算，但这样连线相当复杂，能力所限，我们假设高位都被截断。

则电路如图所示

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201233204.png)

如果将乘法器融合到 ALU 中，我们新增一个未用的：

`ALUctr = 1001`,![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201233541.png)

通过略修改 `OPctr` 的式子，我们可以让 `OPctr` 在 `1001` 时输出 `111`

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201233941.png)

这样，我们就唯一确认了乘法所需的一切。只需要接入 `X * Y ` ，在指令为 `1001` 时候就会解析出 `OPctr=111`，从而输出 MUX 的第 7 条通路，**即乘法结果**。


### 假设在 RV32I 中新增一条指令，导致在 ALU 中增加一个新运算操作，试修改 ALU 设计电路，并通过测试数据进行验证。

和上一道思考题继续。

我们新增指令 `1001`，利用 MUX 的第 7 口，即可完成

测试 `3*4=12`
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201234328.png)


### 分析比较运算使用独立的比较器和使用减法运算通过标志位来实现两种方法的特性。
**使用减法运算通过标志位**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201234528.png)

- 这是我实验中的方案：
- 这种方法，可以复用器件提供的输出接口和信号，比较简洁，运算效率高。
- 例如：无符号比较/有符号比较通过 `SIGctr` 信号做标记减法类型。
	- 有符号比较：**若减完溢出**，或者减完是负数，即 `OF xor SF`，则是 `A < B`
	- 无符号比较：**若减完发生进位**，则是 `A<B`。可以理解为是进位到了虚拟的 33 位的符号位，说明减完变成更高位意义下的负数了。

**使用独立的比较器**
- 这种方案利用了 Logisim 自己的器件，看起来比较简洁，也支持无符号比较和补码比较
- 但是可能增加了组合电路的复杂程度，可能降低 ALU 的效率。
- 同时，也无法给出到底**差**是多少，比较不灵活。
