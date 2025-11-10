名称
- 数字逻辑和计算机组成
- 考核
	- 课后作业、问答：15%
		- [click me check the 3 chapter assignment|400](https://kold.oss-cn-shanghai.aliyuncs.com/d18405c59099b86d65d4996969d22007.png)
		- [[DL-CO-3]]
		- [[DL-CO-4]] 
		- [[DLCO-6]]

	- 六次实验成绩: 35% （实验验收、报告）
		- [[ DLCO-labs-note| About labs]] 
		- [[DLCO-lab1-report]]
		- [[DLCO-lab2-report]]
		- [[DLCO-lab3-report]]


- 世界上第一台电子计算机 ABC（非通用）
	- 不可编程
- 通用电子计算机 ENIAC
	- 能变成
	- 二进制
	- 非存储程序
	- 非冯诺伊曼结构
## 附录

### 电路符号
- Some Note
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250912101512.png)



##  [[计算机如何工作]]

## [[Data in Computer]]
## [[Logic Gate and CMOS]]
## [[DLCO-BooleanAlgebra]]
##  [[组合逻辑电路]]


## [[时序逻辑电路]]

## FPGA 设计和硬件描述语言

可编程逻辑器件（Programmable Logic Device, **PLD**）。
- 介绍可编程逻辑元件
- 介绍 FPGA 和其设计


### 存储器阵列 ROM, RAM

- `ROM`：(Read-only Memory, ROM) **只读存储器。**
	- **非易失性**。
	- 断电也不会消失。
- `RAM`: （Random-access Memory, RAM）**随机存取存储器**
	- **易失性存储器**
	- 静态 RAM
	- 动态 RAM

#### ROM
- **常见实现方式**
	- ROM 存储阵列：MOS **晶体管的有无**来区分存储 `0` 和 `1`
#### RAM
- 静态 RAM (SRAM)
- 动态 RAM (DRAM)

#### 存储阵列表示
`16*8位存储阵列`
- 存储 `16` 个 `8bit` 机器数


### FPGA 设计概述
**FPGA**（Field Programmable Gate Array, FPGA）: **现场可编程门阵列**


## 运算方法和运算部件

### 加法器
- 全加器和半加器
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251015143857.png)
- **串行进位加法器**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251015144433.png)
- 并行进位加法器（CLA）
	- Carry look ahead
	- 通过逻辑方程代入，知道各进位之间无等待，可以相互独立并同时产生。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251015144600.png)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251015145152.png)
- **还有一种折中的办法**
	- 局部（单级）先行进位加法器。

### 带标志的加法器

- `n` 位**带标志**加法器
	- **标识位**用来携带一些信息。
		- 例如溢出 `OF` [[ICS#IA-32 定点寄存器组织|OF寄存器了解]]
			- $OF=C_{n}\oplus C_{n-1}$
			- 原理：`正数加正数得到负数` 和 `负数加负数得到正数` 是溢出的两种，恰好可以用符号位进位 `C_n` 和下一位 `C_n-1` 标识
		- 符号标识位 `SF`
			- $SF=F_{n-1}$
		- 零标识位 `ZF`
			- $ZF=1\iff F=0$
		- 进位标识位 `CF` (Carry)
			- $CF=Cout\oplus Cin$ 
				- `Cout` 是最终输出进位信号
				- `Cin` 是输入的进位信号
				- 如果 `Cout^Cin == 1`，说明中间进位发生了翻转，即产生了新的进位。
	- 我们构造了**带标志加法器**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251015151046.png)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251015151059.png)

- **** 算术逻辑部件（ALU）
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251015151413.png)
	- `ALUop` is actually a `S` in selector circuit. the function of various output is implemented by multi-selector.
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251015152406.png)

### n 位整数加/减运算器



### 无符号整数乘法运算

$P_{i}=2^{-1}(Xy_{i}+P_{i-1})$

- 一个巧妙的 trick
- 将进位巧妙的移入**乘积空间**

- 需要的储存空间：
	- 第一个 `X` 的寄存器
	- 双倍字长的乘积寄存器 `P（Product）+ Y(乘数)`
	- 一个进位 flag `C`


- `32bit` **无符号乘法运算的硬件实现**


- **移位**需要用时钟控制。
	- 引入计数器，计数器运算次数等于乘数 bit 数
### 原码乘法算法
- **用于浮点数尾数乘运算**
- 数值部分：用无符号乘法算法计算
- 符号位：`0 ^ 1 =1`

- 一位乘法：每次只取一位乘法，效率较慢。
- 二位乘法：每次取两位进行运算。
	- 涉及到一些技术细节：
		- 处理符号位
		- 处理新的移位表达式
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251022140244.png)



### 补码乘法运算
- **符号**和**数值**一起运算，算术右移
	- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251022140908.png)
	- **我们可以先不考虑小数点位置**，只需要在最后约定即可。
	- 这样可以简化我们推导。
$$
[x+y]_{补}=\left[ x \times \sum_{i=0}^{n-1}(Y_{i-1}-Y_{i})2^{i}\right]_{补} 
$$
	- 该求和项，实际就模拟了我们手算的过程。我们可以将每一个部分前缀和是部分 `product`，那么
$$
[P_{i+1}]_{补}=[2^{-1}(P_{i}+(Y_{i-1}-Y_{i})x)]_{补},\;(i=0,1,2,\dots,n-1)
$$

- Booth's 算法
- Booth's 算法实际上是在我们上述递推关系的实际应用：
	- 一旦乘数中相邻两位 $Y_{i}Y_{i-1}$ 已经确定，我们就可以让计算机方便的处理，具体遵循以下算法。
	- if $Y_{i}Y_{i-1}=01$, $[P_{i+1}]_{补}=[2^{-1}(P_{i}+x)]_{补}$
	- if $Y_{i}Y_{i-1}=10$, $[P_{i+1}]_{补}=[2^{-1}(P_{i}-x)]_{补}$
	- if $Y_{i}Y_{i-1}=11,00$, $[P_{i+1}]_{补}=[2^{-1}(P_{i}+0)]$
整理为算法，`11，00` 实际是什么都不做。
```
输入：M, Q (均为补码表示，n位)
A = 0
Q_-1 = 0
repeat n times:
    if (Q0 == 0 and Q_-1 == 1):
        A = A + M
    else if (Q0 == 1 and Q_-1 == 0):
        A = A - M
    右移算术移位 (A, Q, Q_-1)
end repeat
结果 = (A, Q)
```


- **用途**：
	- 有符号整数乘法运算。
### 整数的乘运算（溢出）
- 两个 `n` 位乘法结果应该是 `2n` 位
- 如果结果仅保留低 `n` 位，`X*Y` 的高 n 位可以用来溢出判断，规则如下：
	- **无符号**：高 `n` 位全 `0`，则不溢出，否则溢出
	- **带符号**：若高 `n` 位全 `0` 或全 `1` 且等于低 `n` 位的最高位，则不溢出。
		- 即 `进位` 并没有产生低 `n` 位的最高位产生向符号位进位的情况。即没有**数值**信息被进位上去




### 原码除法运算

首先进行取之和大小进行判断。
- 若被除数为 `0`，除数不为 `0`，（或者整数 $|a|<|b|$, $a/b$），商为 `0`，余数为被除数
- 若被除数不为 `0`，除数位 `0`，整数 throw `divider cannot be zero`, 浮点数 `inf`
- 都为 0. 整数发生除法错异常；浮点数，有的机器会产生一个 `quiet NaN`

除法本质是连续的减法。

**减法操作用补码**，所以如果是**无符号数**也需要补一个**符号位**（余数也要留）

- 原码判断够不够减：
	- 得到负数不够减，否则够减

### 恢复余数法

恢复余数法的核心思想是通过**重复的减法和移位操作**来**模拟人工**进行长除法的过程。在每一步骤中，它会尝试用**被除数或当前余数**减去除数，并根据结果来决定：

1. **如果相减的结果是非负的（即余数够减）**：说明商的当前位是 **1**。相减后的结果作为新的余数，且**不需要**恢复。
    
2. **如果相减的结果是负的（即余数不够减）**：说明商的当前位是 **0**。相减后的结果是错误的，需要将**除数加回（即恢复）**，以得到正确的余数，因此得名“恢复余数法”。

总结伪代码：
```
if (R_i >= 0){
	上商1
	R << 1, Q << 1 下一次试商 得到下一次余数 R_{i+1}
	即 R_{i+1} = 2R_i - Y
}
```

我们需要把 **被除数** 进行拓展


## Floating-point number

 **浮点数运算结果**
$A=M_{a}\cdot_{2}^{E_{a}}$
$B=M_{b}\cdot 2^{E_{b}}$

则
$$
A\pm B=(M_{a}\pm M_{b}\cdot2^{-(E_{a}-E_{b})})\cdot{2}^{E_{a}}
$$

### 加减法

四个步骤：
- **对阶**
- **尾数加减**
- **尾数规格化**
- **尾数舍入**

## ISA
**从指令执行周期看指令设计涉及的问题**

```mermaid
graph TD
A[Instruction Fetch]-->B[Instruction Decode]
B-->C[Operand Fetch]
C-->D[Execute]
D-->E[Result Store]
E-->F[Next Instruction]
F-->A
```

参考 PA
[[ICS-PA2 note#更新 PC|如何更新PC]]

- IBM370
**思考**：我们用 16 个通用寄存器来做，**编号需要几位**？
- `4` bit


- **不同指令设计风格**
- CISC
	- Complex Instruction Set Computing
	- 通用寄存器型
- RISC
	- Reduced Instruction Set Computing
	- load/store 型
	- 也用内存，但是必须有专门的访问内存的指令
中断：
- **内部中断**
- **外部中断**
	- **打印机缺纸**



### RISCV 寄存器约定
可以阅读 [[ICS-PA2 note#`riscv` 如何判别 `call` 和 `ret`？|riscv判别`call`和`ret`的办法]]






