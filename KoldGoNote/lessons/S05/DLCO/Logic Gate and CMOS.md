## Section 3
### 逻辑门
- Logic Gate is the most basic numerical circuit
- 具有允许或禁止信号传输的功能。
	- one or more input signal
	- one and only **one** **output** signal
	- **表明逻辑关系**

- **输出变量和输出信号之间的逻辑关系**使用真值表或逻辑表达式来描述
- **真值表**是一个二维表
	- 表头左侧是输入、右侧输出

- 基础
	- 与或非
- 复杂的
- 与非门 NAND
	- $\overline{X\cdot Y}$
	- 与完再非
- 或非门
- 异或，同或
	- `XOR`
		- 输出不同为 1，否则为 0
		- $X\oplus Y$
	- `NXOR`
		- 输出相同为 1，否则为 0
		- $X\odot Y$

### 数字抽象（逻辑采样）

在数字系统中，将一定的电压映射到两个状态：高态和低态，用 0 和 1 表示

- 负载+噪声->可能波动
- 我们定义**未定义区。** 提高容错

- **输入电压**
	- CMOS 晶体管的开关阈值电压决定
	- 输出电压主要由晶体管导通电阻决定。
- **我们对输入电压的要求更高**
- 符号：
	- $V_{IHmin}$ ：确保能被识别为高态的最小输入电压值
	- $V_{ILmax}$：确保能被识别为低态的最大输入电压值
	- $V_{OHmin}$：输出为高态时的最小输出电压值
	- $V_{OLmax}$：输出为低态时的最大输出电压值




- **直流噪声容限 DC noise margin** 是一种对噪声程度的度量，表示多大的噪声会使输出电压被破坏，成为不可被输入端识别的值


### CMOS 晶体管-MOS
- **金属氧化物半导体场效应晶体管**（MOS 晶体管）
- 三极
	- 栅极 gate
	- 源极 source
	- 漏极 drain
- MOS 晶体管
	- n 沟道型 NMOS, (Negative)
	- p 沟道型 PMOS, (Positive)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912102307.png)
- 模型化为一种 **3 端**子压控电阻导体

- **CMOS**晶体管运行
![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20250912102429.png)
- **记忆**
	- NMOS: `V_gs>0`，**栅极为高电平**，导通
	- PMOS: `V_gs <0`，**栅极为低电平**，导通
- **通常觉得接地**。


#### CMOS 结构
- **CMOS** (Complementary Metal-Oxide Semiconductor)
	- **以互补的形式**共用一对 NMOS 和 PMOS 晶体管
	- 栅极和漏极共用，连接 **输入和输出**
		- NMOS source -> 地线 GND
		- PMOS source -> 电源 $V_{DD}$
		- 改变栅极输入电压，只有一个导通，**可以改变输出电压**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912103318.png)

#### CMOS 电路
##### 非门
- **逻辑非**$\lnot$ 是一个单目运算符。
- 一对 CMOS 晶体管实现
- 实际上，一个正常连接CMOS 基本模型就可以当做一个 **非门**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912103723.png)

##### 与非门
- 先与后非
- 真值表

| input1 | input2 | ouput |
| ------ | ------ | ----- |
| 0      | 0      | 1     |
| 0      | 1      | 1     |
| 1      | 0      | 1     |
| 1      | 1      | 0     |
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912104433.png)


##### 或非门

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912104720.png)

---

机器元件层面，**实现与非和或非更加方便**。对于或和与，我们在或非门和与非门基础上再加一个非结构。



#### CMOS-k 输入
- 使用 $k$ 对 NMOS 和 PMOS 晶体管通过串-并联结构，构造一个 $k$ 输入 CMOS 与非门/或非门
- 3 输入与非门包含 3 对 CMOS 晶体管
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912105101.png)

#### 级联

输入端较多的门电路可用输入端较少的门电路 **级联** 而构成

**级联（cascading）** 是指把多个相同或不同的逻辑电路、器件或模块按输入输出顺序依次连接起来，使前一级的输出作为后一级的输入，从而实现更复杂的逻辑功能或扩展电路规模的一种方法。

##### 级联实现与门
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912105412.png)

##### 级联实现缓冲器
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912105518.png)

#### 实现传输门
- 传输门 (transmission gate) 由一对 CMOS 晶体管以及控制信号 EN 构成
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912111226.png)
#### CMOS 电路电气特性
- 转换时间 transition time
	- 逻辑电路的输入信号（或输出信号）从一种状态转换到另一种状态的时间
		- rise time $t_{r}$： from low to high
		- fall time $t_{f}$: from high to low
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912111408.png)
- 我们通过时延设计，减少这个影响。

##### 传播延迟
- $t_{p}$, propagation delay
	- 输入信号变化到引起输出信号变化所需的时间
- 信号通路 signal path
	- **一个特定输入**信号到逻辑元件的特定输出信号所经历的电气通路
- $t_{pHL}$ 从高到低变化的时间延迟
- $t_{pLH}$ 从低到高变化的时间延迟

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250912111747.png)