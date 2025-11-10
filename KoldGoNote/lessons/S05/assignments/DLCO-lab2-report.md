
# 实验 1 组合逻辑部件设计


## 1. 译码器实验
### 实验内容：
根据如图2.1 所示的 3-8 译码器芯片 74X138 的电路原理图，设计一个由反相逻辑门电路构成的 3-8 译码器，并对电路进行仿真测试，以验证电路的功能。

### 基本原理
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019173945.png)


### 设计电路
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251019181340.png)

### 仿真测试

进行仿真测试，
我们给出几条测试的[[图]]片示例
`G1 G2A_L G2B_L C B A`
- `1 0 0 0 0 0`
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019192726.png)
- `1 0 0 0 1 0`
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019192700.png)
- `1 0 0 1 0 0`
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019192822.png)



得到真值表 (**低电平有效**)


| G1 | G2A_L | G2B_L | A | B | C | Y0_L | Y1_L | Y2_L | Y3_L | Y4_L | Y5_L | Y6_L | Y7_L |
|----|--------|--------|---|---|---|-------|-------|-------|-------|-------|-------|-------|-------|
| 1  | 0      | 0      | 0 | 0 | 0 | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| 1  | 0      | 0      | 0 | 0 | 1 | 1 | 1 | 1 | 1 | 0 | 1 | 1 | 1 |
| 1  | 0      | 0      | 0 | 1 | 0 | 1 | 1 | 0 | 1 | 1 | 1 | 1 | 1 |
| 1  | 0      | 0      | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 0 | 1 |
| 1  | 0      | 0      | 1 | 0 | 0 | 1 | 0 | 1 | 1 | 1 | 1 | 1 | 1 |
| 1  | 0      | 0      | 1 | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 0 | 1 | 1 |
| 1  | 0      | 0      | 1 | 1 | 0 | 1 | 1 | 1 | 0 | 1 | 1 | 1 | 1 |
| 1  | 0      | 0      | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 0 |

### 错误记录
未遇到错误

## 2. 8-3 优先级编码器

### 实验内容
优先级编码器原理图，设计一个由逻辑门电路构成的 8-3 优先级编码器，并将编码器输出连
接到一个十六进制数码管，通过数码管的输出显示来验证和测试电路。

### 基本原理
本实验不需要顶层模块设计图
IO：
- `I0 ~ I7`
	- `7` 个输入端，有优先级
- `O0, O1, O2`
	- 输出端，二进制解码

原理图：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019194305.png)


### 设计电路
电路设计图如图所示
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019201254.png)



### 仿真测试

`7`：
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019201316.png)


`4`
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019201327.png)

`0`
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019201344.png)


真值表如图所示：

| I0 | I1 | I2 | I3 | I4 | I5 | I6 | I7 | O0 | O1 | O2 |
|----|----|----|----|----|----|----|----|----|----|----|
| 1  | x  | x  | x  | x  | x  | x  | x  | 0  | 0  | 0  |
| 0  | 1  | x  | x  | x  | x  | x  | x  | 0  | 0  | 1  |
| 0  | 0  | 1  | x  | x  | x  | x  | x  | 0  | 1  | 0  |
| 0  | 0  | 0  | 1  | x  | x  | x  | x  | 0  | 1  | 1  |
| 0  | 0  | 0  | 0  | 1  | x  | x  | x  | 1  | 0  | 0  |
| 0  | 0  | 0  | 0  | 0  | 1  | x  | x  | 1  | 0  | 1  |
| 0  | 1  | 1  | 0  | 0  | 0  | 1  | x  | 1  | 1  | 0  |
| 0  | 0  | 0  | 0  | 0  | 0  | 0  | 1  | 1  | 1  | 1  |


### 错误记录
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019200708.png)
最初版本，弄错了引脚顺序，导致显示屏解读 `001` 为 `100` =4.




## 3 加法器
### 基本原理

串联4 个全加器子电路实现 4 位串行进位加法器。将加数、被加数和和分别连接到 16 进制数码显示管进行验证。实验步骤如下：
### 基本电路

#### 全加器设计

设计电路图如图所示
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019203209.png)


- **`Cout` 转化为与非-与非电路**
	- 进位表达式：`B*Cin + A*Cin + A*B`
	- 转化为与非与非：`~(~(B*Cin) * ~(A*Cin) * ~(A*B))`

- 与非实现的 `Cout`
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019204224.png)

验证电路功能无误。
- 外观：
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019204307.png)
	- 修改的更友好
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019205030.png)


#### 设计加法器

- 思路：各位用 `splitter` 分开交给各个全加器，**每个全加器**的进位输入是低位的进位输出 `Cout`。最终合并各位结果为 `SUM`
- 电路设计图
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019205632.png)
经检验正确

#### 实现补码加减法
- **要求**: 根据 Cin 输入值区分加减法运算，
	- 当 Cin=0 时，执行补码加法运算 `F=X+Y`；
	- 当 Cin=1 时，执行补码减法运算 ` F=X-Y`

只需要当 `Cin=1`，对 `Y` 进行补码（各位取反，末位加一）即可。
- 末位加一的具体实现：将初始 `Cin` **连入最低位全加器来实现**。
- `Cin=0` 取反的具体实现：
	- 可以用一个异或门来方便实现。将 `Cin` 拓展为 `4bit` 和 `Y` 各位做异或

- **测试结果**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019211316.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019211331.png)
- 可以优化第二个数字，显示补码后的数字
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251019211420.png)
- `d` 是十六进制下的补码，表示 `13`，在补码意义下是 `-2` 的意思。
- `3-2=1`

## 4 汉明码校验电路
### 实验内容

实现 `7bit` 汉明码检/纠错电路
### 基本原理

我们使用冗余校验方法。
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251020190909.png)

`Hamming Code` 主要思想：
- 可以用一个 `i bit` 的校验位来进行纠错。一个 `i bit` 校验位可以识别容纳 `2^i-1` 码字，数据位为 `2^i-1 -i bit` 

- 为了方便，我们编码校验位为 `001, 010, 100` 等只有一个 `bit` 为 `1` 的位置
- `7bit` 汉明码的故障字和出错情况的对应关系
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251020191137.png)


- **我们使用偶校验**：即偶数个 `1` 输出 `0`

- 电路原理图：
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251020192006.png)

### 设计电路

#### 3-8 译码器
复用之前设计。

#### 4 位奇偶校验器
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251020202031.png)


#### 组装：汉明码校验

![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251020203229.png)

### 仿真测试
进行测试如图
![|500](https://kold.oss-cn-shanghai.aliyuncs.com/20251020203229.png)
- 功能正常
- 可以纠错![](https://kold.oss-cn-shanghai.aliyuncs.com/20251020220233.png)
	- （**错误记录**）记录了对这部分错误位的理解。

### 错误记录
#### 未初始化常量
最初，我没有初始化 `1` 和 `0` 作为常量输送给 3-8 译码器，这导致其不能正常工作。
在添加 constant 后解决了问题。

#### 其他疑惑

有一个疑惑：`错误位` 是干什么的？
- 我试图解读其为 `P1P2P3`，解读其可以得到出错的编码位，但是其是 `4bit`，那么是什么含义？
- 这并未影响通过 OJ。
- 可能是描绘出现的错误的 `bit`
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251020220033.png)
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251020220233.png)
	- `Input: 1010000`, 可以检测到 `第2 bit` 发生错误，然后纠错完，就是 `0000111`
## 5 桶形移位器

### 实验内容
桶形移位器采用组合逻辑的方式来实现移位功能。
- 有 `n` 位数据输入和 `n` 位数据输出
- 指定移动方向、移动类型（算数/逻辑/循环）和移动位数

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251020220931.png)

### 基本原理
桶形移位器是一个组合逻辑来实现移位功能的器件。可以通过多路选择器来实现。通过不同的控制端，控制输出的端口，然后补 `0` 或移入符号位 来实现逻辑位移或者算数位移。

具体而言：
- 移位方向
	- 左移、右移、不移位
- `A/L`
	- 为 `0`：逻辑
	- 为 `1`：算术

`8` 位桶型移位器可以用三级 `8` bit 选择器实现。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251022102710.png)



### 设计电路

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251022113853.png)
仿照电路原理图，灵活运用 `Splitter` ，完成电路如图。

### 仿真测试
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251022113841.png)

可以正常通过 `OJ` 的所有样例。

- 仿真测试 2
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251022114726.png)

### 错误记录

最初时候，对于 `MUX` 的控制信号 `S` 位数接反了，即把 `shamt` 接到了 `s1` bit，而 `shamt` 应该在 `s0` bit， `Left/Right` 应该在 `s1` bit。该错误导致进行第 2个测试样例，移位没有发生：因为 `Left/Right=0`.


## 思考题

### 修改实验中的加法器电路，生成进位标志 CF、溢出标志 OF、符号标志 SF 和结果为零标志位 ZF。

修改 `lab2.3` 文件
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


接入电路。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251022115917.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251022115937.png)


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251022120026.png)

### 通过带标志位加法器输出（` X=Y`）等于和 `(X<Y)` 小于的信号

`X=Y` 即 `X-Y=0`，即 `X-Y=0 == Cin=1 && ZF =0`
`X<Y` 即 `X-Y<0`，即 `X-Y<0 == Cin=1 && SF =1`

修改电路可得。


### 不使用加法器直接使用逻辑门电路实现 4 位无符号二进制数比较器，输出大于和小于二个结果。

对于无符号数来说，只需要从高位开始比较：
```
for i from highbit to lowbit:
	if(A[i] ^ B[i] != 0){
		if(A[i] == 1){
			return 1; // 大于
		}
		else{
			ruturn -1; // 小于
		}
	}
end loop
return 0; //相等
```

```
output = (A4 & ~B4)
       | ((~(A4 ^ B4)) & A3 & ~B3)
       | ((~(A4 ^ B4)) & (~(A3 ^ B3)) & A2 & ~B2)
       | ((~(A4 ^ B4)) & (~(A3 ^ B3)) & (~(A2 ^ B2)) & A1 & ~B1)

```
- 表达 `A>B`
- 电路图如图：
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027185537.png)
- 对于 `A<B`，仅仅需要对上述电路结果取反即可。不再赘述。
### 如何使用 8 位桶形移位器扩展到 32 位桶形移位器。

设计思想：可以用 4 个 8 位桶形移位器组合形成 32 位。

- **关键思想**：将 `32` bit 拆分为 4 段 `8` bit
```
Input: D[31:0] = {D31..D24, D23..D16, D15..D8, D7..D0}
```

**位移**：
假设我们要左移 `N` 位（0~31）：
1. **高字节位移**
    - 使用 **两个级别的移位**：
        - **一级**：按 8 位块整体移动（0~24 位，步长为 8）
            - 例如，如果 N = 17 位（>16），整个 D31..D16、D15..D0 分块移动 2 个字节
        - **二级**：对每个 8 位块内部再用 8 位桶形移位器完成剩余 `N mod `8 位移位
2. **逻辑结构**
    - **一级多路选择器（MUX）**：控制哪个字节移到哪个位置
    - **二级桶形移位器**：在每个字节内完成 0~7 位精确移位

换言之，我们在外层再加一个多路选择器来控制来控制子 `8` bit 选择器的行为。