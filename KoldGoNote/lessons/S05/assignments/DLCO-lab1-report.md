# 实验 1 基本逻辑部件设计

> 姓名：朱晗
> 学号：`231275036`

## 1. 利用基本逻辑门设计一个 `3` 输入多数表决器
### 实验内容：利用基本逻辑门设计一个 3 输入多数表决器。
### 基本原理：
- **真值表** ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251013200420.png)
- 化简后的表达式为 $F(X,Y,Z)=Y\cdot Z+X\cdot Z+X\cdot Y$
	- 我们需要 `3` 个输入引脚，`1` 个输出，`1`个 `2` 输入与门，`1` 个 `3` 输入或门
### 设计电路
电路图如图所示
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251013200519.png)

电路的颜色包含了一些信息：
- **绿色**（深绿/亮绿）表示线路正常工作，传递着 **0** 或 **1** 的值。
- **蓝色** 和 **灰色** 通常表示线路有问题（值未知或未连接），在电路完成时应该尽量避免。
- **红色** 和 **橙色** 表示**严重错误**（值冲突或位宽不匹配），必须解决才能让电路正常工作。
### 仿真测试
`X=0, Y=0, Z=0, output F=0`
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251013200921.png)
`X=0, Y=1, Z=1, output F=0`
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251013201123.png)
- 剩余测试经检验正确。

## 2.利用 CMOS 晶体管构建两输入或门，并验证其功能
### 基本原理
非门可以由 **或非门级联反相器** 构成。

### 设计电路

设计电路如图
![](https://kold.oss-cn-shanghai.aliyuncs.com/20251013201512.png)
### 仿真测试

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251013201600.png)

![](https://kold.oss-cn-shanghai.aliyuncs.com/20251013201512.png)
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251013201656.png)

不同输入测试下，均正常。
### 记录或门真值表

根据仿真测试的输入输出，得到真值表

| X   | Y   | Z   |
| --- | --- | --- |
| 0   | 0   | 0   |
| 0   | 1   | 1   |
| 1   | 0   | 1   |
| 1   | 1   | 1   |
可见电路实现和标准或门真值表完全一致，是正确的电路。

## 3 利用基本逻辑门和 CMOS 晶体管实现多路选择器，并进行静态冒险检测。
### 基本原理
`2` 选 `1` 多路选择器的逻辑表达式为：$Y=D_{0}\cdot \bar{S}+D_{1}\cdot S$

根据该逻辑表达式：我们需要 `2` 个 `2` 输入与门，`1` 个非门，`1` 个 `2` 输入或门。
- 如果不用 CMOS，则可直接画出。
- 如果用 CMOS，则用 CMOS 模拟非门和与门非门

### 基本电路
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251013203632.png)
- 如果不用 CMOS
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014104304.png)


### 静态冒险检测 ：利用晶体管和传输门实现多路选择器。


我们先输入 `D0=1, D1=1, S=1`，观察输出
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014191330.png)
`CTRL+E` 关闭 `Simulation Enabled`，开启单步仿真。我们改为 `S=0`
以下是过程
- 初始状态
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014191516.png)

- 第 1次单步仿真
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014191534.png)
- 第 2 次单步仿真
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014191601.png)
- 第 3 次单步仿真
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014191616.png)
- 第 4 次单步仿真
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014191636.png)

单步仿真反映了信号在电路中经过逻辑门导致的延迟。
我们进行了 4 次单步仿真，即从输入到输出一共经历了三级逻辑门延迟。


## 4 利用晶体管和传输门实现多路选择器
利用晶体管和传输门的方案：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014192452.png)
验证功能正常。

按照教程，封装 `2-1MUX`
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014193426.png)
- 编辑外观![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014193555.png)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014194029.png)
- 完成级联
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014195925.png)
		- 出现红色，线路链接有错误，经检查是粗心 D1 没有接上。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014195819.png)
- 验证功能正确。完成级联实验。

### 尝试用隧道和引脚功能
熟悉集线器操作。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014203831.png)

## 思考题
### 根据 2 选 1 多路选择器的与 -或电路，替换成与非 -与非电路，并分析两种电路的特性。
2 选 1 多路选择器的与或电路逻辑表达式为：
$Y=D_{0}\cdot \bar{S}+D_{1}\cdot S$
$Y=\overline{{\overline{D_{0}\cdot \bar{S}}}\cdot {\overline{D_{1}\cdot S}}}$

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014205301.png)

用与非-与非电路实现，只有一种门参与，实现通用性更好，且与非在实际电路实现上效率更高，成本更低。
最大门延迟均为三级。

### 实现 4 位二进制数转化为格雷码的转换电路
| 4位自然二进制码 | 4位格雷码 |
| -------- | ----- |
| 0000     | 0000  |
| 0001     | 0001  |
| 0010     | 0011  |
| 0011     | 0010  |
| 0100     | 0110  |
| 0101     | 0111  |
| 0110     | 0101  |
| 0111     | 0100  |
| 1000     | 1100  |
| 1001     | 1101  |
| 1010     | 1111  |
| 1011     | 1110  |
| 1100     | 1010  |
| 1101     | 1011  |
| 1110     | 1001  |
| 1111     | 1000  |
卡诺图化简，得到公式：
$$
G_{i}=B_{i}\oplus B_{i+1}
$$
设二进制为 $B_{3}B_{2}B_{1}B_{0}$，格雷码为 $G_{3}G_{2}G_{1}G_{0}$

实现：
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014211149.png)

### 实现 4 位二进制数的奇偶校验位生成电路。

奇偶校验：
- **偶校验**：总 `1` 的个数为偶数时候，为 `0`，否则为 `1`
- **奇校验**：相反

对于偶校验，只需要对每一位 `ABCD` 做异或，得到 `0` 说明偶数个 `1`
奇校验取反即可。电路：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014212057.png)
