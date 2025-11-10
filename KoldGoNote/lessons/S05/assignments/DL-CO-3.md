### 3. 写出图 3.34 所示电路的对应的逻辑表达式
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251009141436.png)

- Solution:
- 看第一层四个与，输入依次为 $A,B$ | $\bar{A},C$ |, $\bar{A},B,D$, $\bar{B},C,D$ |  $A,\bar{B},{C},\bar{D}$

$$
\begin{align}
F_{1}=(A\cdot B)+(\bar{A}\cdot C)+(\bar{A}\cdot B\cdot D ) \\ \\
F_{2}=(\bar{A}\cdot C)+(\bar{A}\cdot B\cdot D)(\bar{B}\cdot C\cdot D)+(A\cdot \bar{B}\cdot C\cdot \bar{D})
\end{align}
$$

### 4. 假定输出 $F$ 的逻辑表达式为 $\overline{A\cdot B\cdot C\oplus D+\bar{A}+D}$
逻辑电路图如下所示。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251009172600.png)
转化为与-或表达式：
$$
A\bar{B}\bar{D}+A\bar{C}\bar{D}
$$
逻辑电路图如图所示

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251009175044.png)


### 6
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251009143301.png)

列出真值表：

| $I_{0}$ | $I_{1}$ | $I_{2}$ | $I_{3}$ | $I_{4}$ | $I_{5}$ | $I_{6}$ | $I_{7}$ | $O_{0}$ | $O_{1}$ | $O_{2}$ | $Z$ |
| ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | --- |
| 1       | x       | x       | x       | x       | x       | x       | x       | 0       | 0       | 0       | 0   |
| 0       | 1       | x       | x       | x       | x       | x       | x       | 0       | 0       | 1       | 0   |
| 0       | 0       | 1       | x       | x       | x       | x       | x       | 0       | 1       | 0       | 0   |
| 0       | 0       | 0       | 1       | x       | x       | x       | x       | 0       | 1       | 1       | 0   |
| 0       | 0       | 0       | 0       | 1       | x       | x       | x       | 1       | 0       | 0       | 0   |
| 0       | 0       | 0       | 0       | 0       | 1       | x       | x       | 1       | 0       | 1       | 0   |
| 0       | 0       | 0       | 0       | 0       | 0       | 1       | x       | 1       | 1       | 0       | 0   |
| 0       | 0       | 0       | 0       | 0       | 0       | 0       | 1       | 1       | 1       | 1       | 0   |
| 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 1   |

根据真值表，可以列出逻辑表达式


$O_{0}=\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}I_{4}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}I_{5}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}I_{6}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}I_{7}$
$O_{1}=\bar{I_{0}}\bar{I_{1}}I_{2}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}I_{3}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}I_{6}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}I_{7}$
$O_{2}=\bar{I_{0}}I_{1}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}I_{3}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I_{4}}I_{5}+\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}I_{7}$

$Z=\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}\bar{I}_{7}$

用与非门设计，我们对每个逻辑表达式（现在是与-或形式）取两次非，用 de mogen 律化简一次，即可得到与非门方便实现的表达式。

 $O_{0}=\overline{{\overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}I_{4}} \cdot  \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}I_{5}}\\   \cdot    \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}I_{6}} \cdot  \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}I_{7}}}}$
别的同理

$O_{1}=\overline{\overline{\bar{I_{0}}\bar{I_{1}}I_{2}}\cdot \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}I_{3}}\cdot \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}I_{6}}\cdot \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}I_{7}}}$

$O_{2}=\overline{ \overline{\bar{I_{0}}I_{1}\cdot \bar{I_{0}}\bar{I_{1}}\bar{I_{2}}I_{3}}\cdot \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I_{4}}I_{5}}\cdot \overline{\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}I_{7}}}$

$Z=\bar{I_{0}}\bar{I_{1}}\bar{I_{2}}\bar{I_{3}}\bar{I}_{4}\bar{I}_{5}\bar{I}_{6}\bar{I}_{7}$

设计逻辑电路图如下


![image.png|700](https://kold.oss-cn-shanghai.aliyuncs.com/20251010200322.png)

- **优先级顺序**
	- $I_{0}>I_{1}>I_{2}>I_{3}>I_{4}>I_{5}>I_{6}>I_{7}$


## 7 已知一个组合逻辑电路的功能可用图 3.35 的真值表描述，分别用下列器件实现该电路.
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010200813.png)

### （1）一个 8 路选择器


![3102eecc693bacc877d9e2b1510a726b.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/3102eecc693bacc877d9e2b1510a726b.png)

### （2）一个 4 路选择器和一个非门

![462cfd5df5683ce8fcdc8543e2c07b92.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/462cfd5df5683ce8fcdc8543e2c07b92.png)

### （3）一个 2 路选择器和两个逻辑门
![5e620539b7a4c30a1aa632ae3b9decdb.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/5e620539b7a4c30a1aa632ae3b9decdb.png)

![de0ab168c4838d4333fa3dc13deb9d45.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/de0ab168c4838d4333fa3dc13deb9d45.png)

## 9
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010204559.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010204606.png)

### (1) 利用无关项进行化简，写出函数 $F$ 的最简逻辑表达式

画出卡诺图表格
![5e7523fd79ee691f5475008d867e5ac5_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/5e7523fd79ee691f5475008d867e5ac5_720.png)

化简结果：
$F=A\cdot B+A\cdot \bar{D}+A\cdot C$

### （2）逻辑电路图
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010211747.png)

### （3）竞争冒险
不存在竞争冒险。因为是（与-或）表达式，且各变量不存在逻辑相反的变量，故不会出现逻辑相反的变量冲突冒险的问题。

## 11 传输延迟问题
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251010212155.png) 

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010212230.png)

a. 初始电路
- `T_pd = 40 + 55 =95 ps, T_cd = 25 ps`
b. 加入反相器的电路
- `T_pd = 40 + 15 + 15 + 55 = 125 ps, T_cd = 10 + 10 + 25 = 45 ps`
c. 使用反向输出端和反向输入端的电路
- **注意反向输入端与门是或非门等效电路，反向输入端或门是与非门的等效电路**
	- $\bar{A}\cdot \bar{B} =\overline{A+B},\bar{A}+\bar{B}=\overline{AB}$
- `T_pd = 30 + 30 = 60ps , T_cd = 10 + 25 = 35 ps`

