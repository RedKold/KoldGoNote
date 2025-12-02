# OLS
### D-W 检验

## OLS 的 BLUE 条件
[[Gauss-Markov假设]] 

## 矩阵表示
- OLS
- 最小二乘法的矩阵表示
- $Y=\beta_{1}+\beta_{2}X_{2}+\beta_{3}X_{3}+L+\beta _kX_{k}+u$
- 矩阵表示
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250916081349.png)

- $$\boldsymbol{\text{Var}(\hat{\boldsymbol{\beta}} \mid \mathbf{X}) = \sigma^2 (\mathbf{X}'\mathbf{X})^{-1}}$$
- $\mathbf{\hat{\beta}}=(\mathbf{X'X})^{-1}\mathbf{X}'\mathbf{y}$
- $\mathrm{y}$ 是唯一的随机变量。随机性来自 $u$
- **解释变量是外生的**。才可以推出
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250916083218.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250916083406.png)

### 检验

#### T **统计量**
- **$t$ 检验**
	- $t-统计量=\dfrac{\hat{\beta}_{i}}{SE(\hat{\beta}_{i})}$
	- **$SE(\hat{\beta}_i)$ (Standard Error of the Estimate):** $\hat{\beta}_i$ 的**标准误差**。它衡量了 $\hat{\beta}_i$ 的估计值在不同样本中变动的程度（即估计值的精度）。
#### p 值
- $p$ 值
	- 是 $t$ 统计量根据 $t$ 分布（t-distribution）转换而来
	- 在假设**原假设**（$\beta=0$, for example）是真实的情况下，观测到当前这个 t 统计量（**或更极端的结果**）的概率。
		- 在 $t$ 分布图像上，
		- **如果 p 值≤α：** 拒绝原假设。你的系数是**统计显著**的。
		- **如果 p 值>α：** 不拒绝原假设。你的系数是**统计不显著**的。
#### F 检验
- $F$ -检验：检验多个假设是否成立
	- ![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20250916081924.png)
	- **restricted** and **unrestricted**，前者的残差平方和更大
		- 这是因为无约束的解可以在整个参数空间中找到全局最小值，而有约束的解只能在一个受限的子空间中寻找最小值。除非无约束的最优解恰好满足 β1​+β2​=2（for a example） 这个条件，否则加上约束都会导致拟合效果变差，残差平方和因此增大。
	- **放宽约束条件，我们会多获得代价**。
		- 为此我们引入影子价格：
	- 影子价格（Shadow Price），又称对偶价格（Dual Price），是一个经济学和优化理论中的重要概念。它衡量了在**一个最优解**下，将某个约束条件放宽一个单位所带来的**目标函数值的变化量**。

简单来说，影子价格回答了一个核心问题：**为了获得某个稀缺资源，你愿意多花多少代价？或者说，多一个单位的某个资源，能给你带来多少额外收益？**

---

 **核心概念**

1. **约束条件（Constraints）**：在现实世界的经济活动或优化问题中，我们总会面临资源限制。例如，生产一个产品需要工人（劳动力约束）、原材料（材料约束）和机器（设备约束）。
    
2. **目标函数（Objective Function）**：你想要最大化（如利润、收益）或最小化（如成本、风险）的函数。
    
3. **最优解（Optimal Solution）**：在所有约束条件下，能使目标函数达到最优的那个方案。


- **当我损失一个自由度**，我会失去对一个变量的观测


## 离散变量
- 选择变量，指示变量： 0/1

- **二元选择模型**
	- Probit and Logit
$$
p_{t}=F(y_{t})=\frac{1}{\sqrt{ 2\pi }}\int_{-\infty}^{y_{t}}e^{-v^{2}/2}\mathrm{d}v \tag{Probit}
$$
$$
p_{t}=F(y_{t})=\frac{1}{1+e^{-y_{t}}}\tag{Logit}
$$
- where  $y_{t}=\alpha+\beta x_{t}+u_{t}$

- Logit bi  Probit 更适合二元选择建模

### 多元线性回归系数估计的标准误

标准误由下面的的协方差矩阵中的对角线元素开方得到。


### 极大似然估计 (MLE)
极大似然估计（MLE， Maximum Likelihood Estimation）

- OLS：使得模型尽可能最好的拟合数据
- MLE： 模型能以最大的可能产生样本数据（样本出现的概率最大）
- 考虑回归模型 
	-  $y_{t}=X_{t}\beta+u_{t}$
- 其中，$X_{t}=(x_{1,t},x_{2,t},\dots,x_{k,t})$, $\beta=(\beta_{1},\beta_{2},\dots,\beta_{k})'$

- 设 $u_{t}$ 为相互独立且服从**密度函数**为 $f(\nu ;\eta )$, 其中 $\eta$ 为该分布函数的形态参数向量
	- 例如，标准正态分布假设下，$\eta=(0,\sigma^{2})$；在标准学生-t 分布假设下，$\eta$ 表示自由度；等等
- 样本数据以最大可能出现的 $(\beta,\eta)$，**应能极大化下面的联合概率密度**
$$
\max_{\beta,\eta}L(\beta,\eta)=\prod_{t=1}^{T}l_{t}(\beta,\eta)
$$
其中, $l_{t}=f(u_{t};\eta)=f(y_{t}-X_{t}\beta;\;\eta)$

**两边取对数**，令 $\ell=\ln l_{t}$, 极大化问题转化为

$$
\max_{\beta,\eta}L^{*}(\beta,\eta)=\sum_{t=1}^{T}\ell_{t}
$$
- $L^{*}(\beta,\eta)$ 称为 **对数似然函数**
	- **对数化连乘为连加**
- 采用**极大似然原理**，在已知扰动项分布函数的情况下，关于参数 $(\beta,\eta)$ 极大化 $L^{*}(\beta,\eta)$ 而得到的 $(\hat{\beta},\hat{\eta})$，称为参数 $(\beta,\eta)$ **的极大似然估计量 （值）**

 **MLE 量**的性质
- 一致性
	- 渐进一致性 $(\hat{\beta},\hat{\eta})\to(\beta,\eta)$
- 正态性
	- 依分布收敛到正态分布， $\sqrt{ T }(\hat{\beta}-\beta)\to_{d} N(0,F^{-1})$
	- $F$ 为费雪信息矩阵 (Fisher's Information Matrix)
$$
F=\lim_{ T \to \infty } E\left( \frac{\partial^{2}\ell_{t}}{\partial \beta \partial \beta'} \right) =\lim_{ T \to \infty }T\mathrm{E}\left( \frac{{\partial \ell_{t}}}{\partial \beta} \cdot \frac{{{\partial \ell_{t}}}}{\partial \beta'}\right)  
$$
- 样本点的 MLE 函数 $\ell_{t}$ 在极值处的海塞矩阵 (Hessian matrix)，或近似计算为梯度向量的外积和
## 计数因变量


实践中我们经常会讨论一系列用于事件发生次数进行建模的方法。
- 商业银行贷款逾期次数
- 公司研发部门的专利产出个数
- 证券分析师评级下调/上调次数，正向/负向/积极的次数

### 泊松计数模型
[[常用随机变量分布类型总结#1. 泊松分布]]

泊松回归原理是使用一系列**解释变量**对**计数被解释变量的均值**进行回归
$$
\lambda=\beta_{0}+\beta_{1}x
$$
- **核心思想**：将柏松分布的**平均到达强度**$\lambda$ 用一个**线性模型**刻画
- **但是**泊松分布的定义表明平均到达强度 $\lambda$ 不可能为负数。
- 上述关系无法保证非负性，**我们修正设定**
$$
Prob(Y=y_{i} |X_{i})=\frac{e^{\lambda_{i}}\lambda _{i}^{y_{i}}}{y_{i}!},\;y_{i}=0,1,2,3,\dots
$$
	- where $\lambda_{i}=e^{\mathrm{x}_{i}'\beta}$, 即 $\ln\lambda_{i}=\mathrm{x}_{i}'\beta$ 
	- 这样我们保证了 $\lambda$ 的非负性
	- **回忆泊松分布**, [[常用随机变量分布类型总结#1. 泊松分布]]，上面的概率即柏松分布的 $Y=y_{i}$ 的**概率质量函数**（在某一点取值的概率）。（它告诉我们，如果平均数是 $\lambda_{i}$  ​，那么观察到 $y_{i}$ ​ 次的概率是多少。）
- 泊松回归不是直接对计数变量 Y 进行回归，而是对它的均值 λ 进行回归。它使用一个线性组合来解释这个均值

- **当我们总体抽取得到一个样本**，联合概率密度为
$$
f(y_{1},y_{2},\dots,y_{n}\;|\;X)=\prod_{i=1}^{n}{\frac{e^{\lambda_{i}}\lambda_{i}^{y_{i}}}{y_{i}!}}
$$

**加一个扰动项**


### **虚拟变量**
- 一定会考 #必考 
现实中常见的定性自变量，如：性别、学历、民企/国企、央企/地方企业、金融/非金融企业等；通过构造**虚拟变量**将进行“量化”。

### 定性变量的量化

虚拟变量（Dummy Variables），也叫哑变量，是回归分析中用来表示定性数据（如性别、地区、季节等）的一种方式。在模型中引入虚拟变量时，主要有两种方法：**加法方式**和**乘法方式**。

---

#### **加法方式：改变截距项**

加法方式引入虚拟变量，是最直接、最常见的方法。它通过改变模型的**截距项**来反映不同类别之间的均值差异。

- **模型形式**：
- $Y=\beta_{0}+\beta_{1}X_{1}+\delta D+\epsilon$
    - Y 是因变量。
    - X1​ 是一个定量解释变量。
    - D 是一个虚拟变量（比如，男性 D=0，女性 D=1）。
- **含义**：
    - 当 D=0 时（男性），模型为：Y=β0​+β1​X1​+ϵ
    - 当 D=1 时（女性），模型为：Y=(β0​+δ)+β1​X1​+ϵ
    
    这里的 δ 就表示女性群体的截距项比男性群体多出的那部分。它衡量了在 X1​ 保持不变的情况下，不同群体之间因变量的**平均差异**。
    
- **图形解释**：在坐标系中，加法方式相当于在**保持斜率不变**的情况下，将整条回归线**向上或向下平行移动**。
    

---

#### **乘法方式：改变斜率**

乘法方式引入虚拟变量，通常用于考察不同类别对定量变量的**影响程度是否不同**。它通过在模型中引入虚拟变量和定量变量的**交互项**来实现。

- **模型形式**：
$$
y_{i}=\alpha+\beta_{i}x_{i}+\gamma D_{i}x_{i}+u_{i}
$$
- $\delta$ 仍然是加法项，表示截距的差异。
- $\gamma$ 是**交互项**(Interaction Effect) $D\cdot x_{i}$ ​ 的系数。
	- 如果 $x_{i}$ 是一个向量，则有一系列 $x_{1i},x_{2i},x_{3i},\dots,x_{ki}$。与
- **$i$** 表示第 $i$ **次观测**
- **含义**：
    
    - 当 D=0 时（男性），模型为：Y=β0​+β1​X1​+ϵ
        
    - 当 D=1 时（女性），模型为：Y=(β0​+δ)+(β1​+γ) X1​+ϵ
        
    
    这里的 γ 就表示女性群体的回归线**斜率**比男性群体多出的那部分。它衡量了不同群体因变量随 X1​ 变化的**边际效应差异**。
    
- **图形解释**：在坐标系中，乘法方式允许不同群体的回归线拥有**不同的斜率**。这不仅可以改变截距，还可以让两条线相交。
    

---

#### **总结与应用场景**

|特点|加法方式|乘法方式|
|---|---|---|
|**引入方式**|虚拟变量本身|虚拟变量与定量变量的交互项|
|**影响**|改变截距项|改变斜率|
|**回答问题**|“不同群体的平均水平是否有差异？”|“不同群体的边际影响（斜率）是否有差异？”|
|**应用场景**|衡量性别对工资的平均影响，地区对房价的平均影响。|衡量教育年限对男女工资的影响是否不同，或季节对商品价格的影响是否不同。|

在实际分析中，你也可以将两种方式结合起来使用，即同时包含虚拟变量和交互项，以全面考察定性变量对模型的影响。


#### **混合形式**

$$
Y_{i}=\beta_{0}+\beta_{1}X_{1i}+\delta D_{i}+\gamma(D_{i}\cdot X_{1i})+\epsilon_{i}
$$ 
- $\gamma$ 是交互项 $D_{i}X_{1i}$ 的系数
- $\epsilon_{i}$ 是残差 (Residuals)
- $D_{i}=0$
	- $\mathbf{Y_{i}=\beta_{0}+\beta_{1}X_{i1}+\epsilon_{i}}$
	- **此是样本回归线（坐标系上）或者样本回归函数**
	- 截距项：$\beta_{0}$
	- 斜率（$X_{1i}$ 的边际效应）：$\beta_{1}$
- $D_{i}=1$
	- $\mathbf{Y_{i}}=(\beta_{0}+\delta)+(\beta_{1}+\gamma)\mathbf{X_{i1}}+\epsilon_{\mathbf{i}}$
	- 截距项：$\beta_{0}+\delta$
	- 斜率：$\beta_{1}+\gamma$

[[Oller-Pakes 研究]]



## Chow Test
- 本质是 F 检验
- 全样本回归，获得 $RSS$

$$
r=\beta_{0}+\beta_{1}\delta_{1}+\beta_{2}\delta_{2}+\dots+\beta_{k}\delta_{k}+\varepsilon
$$


	- 有约束的残差平方和
- 再跑两个不同时期的样本 (`sub-sample1, sub-sample2)`，获得 $RSS_{1}$ 和 $RSS_{2}$

$$
\begin{align}
r=\phi_{0}+\phi_{1}\delta_{1}+\dots+\phi_{k}\delta_{k}+u\tag{subsample 1} \\


r=\varphi_{0}+\varphi_{1}\delta_{1}+\dots+\varphi_{k}\delta_{k}+u\tag{subsample 2} \\
\end{align}
$$


 **重要观点**
	- 基于 **全样本** 回归的 `RSS` 可以理解为 **子样本** 回归在 $\phi_{0}=\varphi_{0},\phi_{1}=\varphi_{1}\dots$ 得到的
	![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250930094232.png)

# 说说自相关

## 自相关问题
**自相关**：两个解释变量之间存在相关性，即 $Cov(x_{i},x_{j})\neq 0$
### 自相关不影响无偏性
这是一个经典的计量经济学问题，涉及到 OLS 估计量在违反**同方差/无自相关**假设时的性质。

对于包含**一阶自相关（First-Order Autocorrelation）** 的模型，即误差项 $\epsilon_t$ 遵循 $\epsilon_t = \rho \epsilon_{t-1} + u_t$ 且 $|\rho| < 1$ 的情况，**OLS 估计量 $\hat{\boldsymbol{\beta}}$ 仍然是无偏的（Unbiased）**。

#### 证明无偏性的核心逻辑

要证明 OLS 估计量 $\hat{\boldsymbol{\beta}}$ 是无偏的，我们只需要证明其期望值等于真实的总体参数 $\boldsymbol{\beta}$，即：

$$E[\hat{\boldsymbol{\beta}}] = \boldsymbol{\beta}$$

OLS 估计量的矩阵形式是：

$$\hat{\boldsymbol{\beta}} = (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\mathbf{y}$$

将模型 $\mathbf{y} = \mathbf{X}\boldsymbol{\beta} + \boldsymbol{\epsilon}$ 代入：

$$\hat{\boldsymbol{\beta}} = (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'(\mathbf{X}\boldsymbol{\beta} + \boldsymbol{\epsilon})$$

$$\hat{\boldsymbol{\beta}} = (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\mathbf{X}\boldsymbol{\beta} + (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\boldsymbol{\epsilon}$$

$$\hat{\boldsymbol{\beta}} = \boldsymbol{\beta} + (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\boldsymbol{\epsilon}$$

现在我们求 $\hat{\boldsymbol{\beta}}$ 的期望值：

$$E[\hat{\boldsymbol{\beta}}] = E[\boldsymbol{\beta} + (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\boldsymbol{\epsilon}]$$

$$E[\hat{\boldsymbol{\beta}}] = \boldsymbol{\beta} + E[(\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\boldsymbol{\epsilon}]$$

**无偏性的关键：**

要使 $E[\hat{\boldsymbol{\beta}}] = \boldsymbol{\beta}$ 成立，必须保证第二个项的期望值为零：

$$E[(\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\boldsymbol{\epsilon}] = \mathbf{0}$$

根据**经典假设**：

1. 解释变量 $\mathbf{X}$ 是**非随机**的（或条件于 $\mathbf{X}$）。
2. 误差项的**期望值**为零：$E[\boldsymbol{\epsilon}] = \mathbf{0}$。
由于 $(\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'$ 只包含非随机的 $\mathbf{X}$，我们可以将期望运算带入 $\boldsymbol{\epsilon}$：
$$E[(\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}'\boldsymbol{\epsilon}] = (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}' E[\boldsymbol{\epsilon}] = (\mathbf{X}'\mathbf{X})^{-1}\mathbf{X}' \cdot \mathbf{0} = \mathbf{0}$$
#### 结论

**一阶自相关（或任何形式的自相关）的存在，并不会影响 OLS 估计量的无偏性。**

**为什么？**

因为 OLS 估计量的无偏性**只依赖于**以下两个条件（在 $\mathbf{X}$ 非随机的假设下）：

1. **扰动项的期望为零：** $E[\boldsymbol{\epsilon}] = \mathbf{0}$。
    
2. **解释变量与扰动项不相关：** $E[\mathbf{X}'\boldsymbol{\epsilon}] = \mathbf{0}$。
    

**自相关（Autocorrelation）** 是指 $\text{Cov}(\epsilon_t, \epsilon_{t-s}) \neq 0$，它违反的是 OLS 的**同方差/无自相关**假设。这个假设是保证 OLS 具备**效率（Efficiency）** 所必需的。

#### 总结自相关的影响

| **统计性质**               | **影响**   | **结论**                                                             |
| ---------------------- | -------- | ------------------------------------------------------------------ |
| **无偏性 (Unbiasedness)** | **不受影响** | OLS $\hat{\boldsymbol{\beta}}$ **仍然无偏**。                           |
| **一致性 (Consistency)**  | **不受影响** | OLS $\hat{\boldsymbol{\beta}}$ **仍然一致**。                           |
| **效率 (Efficiency)**    | **受到影响** | OLS $\hat{\boldsymbol{\beta}}$ **不再是最有效率**的（不再是 BLUE），因为它的方差不是最小的。 |
| **统计推断**               | **严重影响** | 报告的标准误**有偏**（通常低估），导致 $t$ 检验和 $F$ 检验的结论**不可信**。                    |

## 自相关检验
### DW 检验（Durbin-Watson Test）总结

**DW 检验**是计量经济学中用于检测**时间序列数据回归模型**残差中是否存在**一阶自相关（First-Order Autocorrelation）**的统计方法。
#### 1. 检验目的
- **核心假设：** OLS 模型的经典假设之一是误差项是独立且不相关的。
- **DW 作用：** 检验**残差 $\epsilon_t$ 与其前一期残差 $\epsilon_{t-1}$ 是否相关。**
#### 2. DW 统计量的计算与范围
DW 统计量 $d$ 的值总是在 **0 到 4 之间**。

$$d = \frac{\sum_{t=2}^{T} (e_t - e_{t-1})^2}{\sum_{t=1}^{T} e_t^2}$$
#### 3. DW 值的核心解读

|**DW 值范围**|**残差关系**|**自相关类型**|**OLS 影响**|
|---|---|---|---|
|**$d \approx 2$**| $e_t$ 和 $e_{t-1}$ 不相关|**无自相关**（理想状态）|OLS 是 BLUE|
|**$0 \le d < 2$**| $e_t$ 和 $e_{t-1}$ 同号倾向|**正自相关**（最常见）|标准误被低估，统计推断不可信|
|**$2 < d \le 4$**| $e_t$ 和 $e_{t-1}$ 异号倾向|**负自相关**（较少见）|标准误被低估，统计推断不可信|

#### 4. 统计推断：为什么 DW 值需要接近 2？

自相关问题虽然**不影响 OLS 估计量的无偏性和一致性**，但它严重违反了保证 OLS 具有**最小方差（效率）** 的第五个假设（球形扰动项）。

- **后果：** 存在自相关时，OLS 计算出的标准误是错误的（通常是低估），导致系数的 $t$ 统计量被人为放大。这会让你**错误地拒绝原假设**（犯第一类错误），认为原本不显著的变量是显著的。
    

#### 5. 局限性

DW 检验有两个主要限制：

1. **仅适用于一阶自相关。** 无法检测二阶或更高阶的自相关。
    
2. **不适用于模型中包含滞后被解释变量（如 $y_{t-1}$）的情况。** 此时 DW 检验结果有偏，应使用 **Durbin 的 $h$ 检验**或 **Breusch-Godfrey LM 检验**。

# 异方差性
**异方差性**（Heteroskedasticity）是计量经济学和统计学中对线性回归模型的一个关键假设的违反。

简单来说，它的意思是：**模型误差项的方差随着解释变量（自变量）的不同取值而发生变化，不再是一个常数。**

---

### 1. 核心定义

在 OLS（普通最小二乘法）回归模型中，一个基本假设是**同方差性（Homoskedasticity）**。

- 同方差性（理想状态）：
    
    $$\text{Var}(\epsilon_i \mid \mathbf{X}) = \sigma^2$$
    
    这意味着，对于所有观测值 $i$，误差项 $\epsilon_i$ 的方差都是一个常数 $\sigma^2$。残差的散布是均匀的。
    
- 异方差性（问题状态）：
    
    $$\text{Var}(\epsilon_i \mid \mathbf{X}) = \sigma_i^2$$
    
    这意味着，对于不同的观测值 $i$，误差项的方差是不同的 $\sigma_i^2$。残差的散布是不均匀的。
    

### 2. 直观理解

你可以想象一个关于家庭收入（解释变量 $X$）对家庭消费支出（因变量 $Y$）的回归模型：

- **低收入家庭：** 他们的消费支出往往比较稳定，因为大部分收入都用于必需品，误差项（$Y$ 与预测值之间的偏差）的方差**较小**。
    
- **高收入家庭：** 他们的消费支出波动性很大，有些人节俭，有些人挥霍，误差项的方差**较大**。
    

在这种情况下，误差项的方差随着解释变量（收入）的增大而增大，模型就存在**异方差性**。

### 3. 异方差性的后果

当模型中存在异方差性时，OLS 估计量 $\hat{\boldsymbol{\beta}}$ 仍然具备以下性质：

1. **无偏性（Unbiasedness）：** $E[\hat{\boldsymbol{\beta}}] = \boldsymbol{\beta}$
    
2. **一致性（Consistency）：** $\hat{\boldsymbol{\beta}}$ 在大样本下趋向于真实值。
    

**但是，它会破坏 OLS 估计量的 BLUE（最佳线性无偏估计量）性质：**

1. **不具效率性（Inefficient）：** OLS 估计量不再是最小方差的估计量（不再是 BLUE），存在其他更有效的估计方法（如加权最小二乘法 WLS）。
    
2. **统计推断无效：** **这是最严重的问题。** OLS 计算的**标准误**是**有偏且不一致**的（通常是低估）。这会导致：
    
    - **$t$ 统计量**被人为放大。
        
    - **$p$ 值**被人为缩小。
        
    - 研究者更有可能**错误地拒绝原假设**，从而得出系数显著的错误结论。
        

### 4. 解决异方差性

一旦通过 **怀特检验**、**布鲁施-佩根检验（Breusch-Pagan Test）**等方法检测到异方差性，常用的解决办法是：

1. **使用异方差稳健标准误（Robust Standard Errors）：** 也称 Huber-White 标准误。这是最常用的方法，它在不改变系数估计值的情况下，纠正了标准误，使统计推断重新有效。
    
2. **使用加权最小二乘法（Weighted Least Squares, WLS）：** 如果异方差的具体函数形式已知，WLS 通过对具有较小误差方差的观测值赋予更大的权重，来获得更有效的估计量。

### 检验异方差性：怀特检验
**怀特检验**（White Test）是计量经济学中用来检测回归模型残差是否存在**异方差性（Heteroskedasticity）**的一种广泛使用的、**通用（General）**的检验方法。

它由经济学家哈尔伯特·怀特（Halbert White）于 1980 年提出。

---

#### 1. 检验的核心目的

- **核心假设：** OLS 的第五个假设要求误差项具有**同方差性（Homoskedasticity）**，即 $\text{Var}(\epsilon_i \mid \mathbf{X}) = \sigma^2$（误差方差是一个常数）。
    
- **怀特检验目的：** 检测这个同方差假设是否被违反，即误差方差是否随解释变量的值而变化。
    

##### 异方差的影响

与自相关类似，异方差**不影响 OLS 估计量的无偏性和一致性**，但它会导致 OLS 估计量的标准误**不一致**（通常低估），从而使统计推断（$t$ 检验和 $F$ 检验）不可信。

#### 2. 怀特检验的步骤（辅助回归）

怀特检验的精妙之处在于它不需要事先假设异方差的具体函数形式，它通过一个辅助回归来完成检测：

##### 步骤 1：原始 OLS 估计

首先，对原始模型进行 OLS 估计，并得到残差 $\hat{e}_i$。

$$y_i = \beta_0 + \beta_1 x_{i1} + \dots + \beta_k x_{ik} + \epsilon_i$$

##### 步骤 2：构造辅助回归

使用**残差的平方** $\hat{e}_i^2$ 作为新的因变量，并对以下一组解释变量进行回归：

$$\hat{e}_i^2 = \delta_0 + \delta_1 x_{i1} + \dots + \delta_k x_{ik} + \delta_{k+1} x_{i1}^2 + \dots + \delta_{2k} x_{ik}^2 + \dots + \text{交叉乘积项} + \text{v}_i$$

简而言之，就是将 $\hat{e}_i^2$ 对**所有原始解释变量、它们的平方项，以及所有两两交叉乘积项**进行回归。

##### 步骤 3：计算统计量

从辅助回归中获取**决定系数 $R^2$** 和**样本容量 $n$**。怀特检验统计量 $LM$ 为：

$$\boldsymbol{LM = n \cdot R^2}$$

该统计量渐近服从自由度为 $m$ 的**卡方分布**($\chi^2$)，其中 $m$ 是辅助回归中解释变量的数量（不包括截距项）。

#### 3. 假设与结论

|**假设**|**描述**|**决策标准**|**最终结论**|
|---|---|---|---|
|**原假设 ($H_0$)**|**模型存在同方差性** ($\delta_1 = \delta_2 = \dots = 0$)|**$p \text{ 值} > 0.05$**|**不拒绝 $H_0$，无异方差**|
|**备择假设 ($H_1$)**|**模型存在异方差性**（至少一个 $\delta$ 不为零）|**$p \text{ 值} \le 0.05$**|**拒绝 $H_0$，存在异方差**|

#### 4. 怀特检验的优点与局限性

##### 优点（通用性）

- **通用性强：** 怀特检验是**最通用**的异方差检验之一，因为它不需要事先指定异方差的具体函数形式。它可以检测几乎任何形式的异方差。
    

##### 局限性

- **消耗自由度：** 辅助回归中包含大量解释变量（原始变量、平方项和交叉乘积项）。如果原始模型中解释变量很多，辅助回归会消耗**大量自由度**，因此怀特检验对**小样本**数据不友好。
    
- **联合检验：** 怀特检验也是一个**模型设定误判检验**。如果拒绝了 $H_0$，可能不仅是因为异方差，也可能因为模型函数形式设定有误（例如，应该用 $x^2$ 而不是 $x$）。
    

#### 5. 解决异方差的方案

如果怀特检验拒绝了 $H_0$，最常用的解决方法是使用 **稳健标准误（Robust Standard Errors）**，也称为 **异方差一致标准误（Heteroskedasticity-Consistent Standard Errors）**，通常称为 **White 标准误**或 **Huber-White 标准误**。这种方法在不改变系数估计值 $\hat{\boldsymbol{\beta}}$ 的情况下，纠正了标准误的低估问题，从而使统计推断有效。
# 面板数据回归
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014080650.png)

- **为什么使用面板数据**
	- **有助于解决遗漏变量问题**
	- **遗漏变量**
		- 异质性 heterogeneity
		- 不随时间而改变 time invariant
	- **个体动态信息**
		- 横截面、时间两个维度


## 面板数据简单估计
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014081340.png)

- **混合回归**：Pooled Ordinary Least Squares, POLS

## 固定效应模型
- **个体固定模型**
	- 若 $u_{i}$ 与某个**解释变量相关**，则为“固定效应”(Fixed Effects, 简记 FE)
	- **此时直接混合回归**，OLS 不一致，因为复合扰动包括 $(u_{i}+\varepsilon_{it})$ 两部分
		- **OLS 失败的原因：** OLS 模型的经典假设要求**扰动项与所有解释变量不相关**。
		- 扰动项与解释变量相关，导致了**内生性问题**。在这种情况下，直接对全样本进行**混合回归（Pooled OLS）** 会导致 β 的估计量是**不一致**（Inconsistent）的
**面板数据的基本结构**：
- $y_{it}=\mathrm{x}_{it}'\beta+u_{i}+\varepsilon_{it}$

### 组内去平均法 (Within Transformation)


### 最小二乘虚拟变量法 (Least Square Dummy Variable, LSDV)

- 我们考虑为每一个个体异质性 $u_{i}$ 添加 `N-1` 个虚拟变量。
	- 为什么不是 `N` 个？
		- 完全多重共线性，导致无法估计。
- [[完全多重共线性]]
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014084349.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014084817.png)

#### 考虑缺点

- **虚拟变量**：我们引入大量虚拟变量，就会增加估计的量，自由度就会下降。
- **估计误差**增大：
	- 导致 $t$ **统计量变小**
	- 导致 $p$ 值变大
	- 导致我们更可能接受**原假设**。$p\geq \alpha$
	- 导致我们更可能犯第**二类错误**（纳伪）（**接受了错误的假设**）


## 考虑时间
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251014094845.png)
- **双向固定效应**，引入双向。
	- `100个公司，10年`：需要 `99+9=108` 个虚拟变量。
- 引入 $\lambda_{t}$ 表示时间上的。


## 随机效应模型

**个体效应 $u_{i}$ 和所有解释变量 $(x_{it},z_{i})$ 均不相关** （OLS 一致但不有效），则称为 **随机效应模型**(Random effects model, **RE**)
$$
y_{it}=x_{it}'\beta+z_{i}'\delta+u_{i}+\varepsilon_{it}
$$
扰动项包含 $(u_{i}+\varepsilon_{it})$ 两个成分，OLS 不有效（但是**一致**（consistency： 一致指的是估计值大样本下无偏））。
- 不有效 (inefficient)
	- **估计的方差变大**。



## 固定效应还是随机效应模型？

### 豪斯曼检验 (Hausman)

- **原假设**：随机效应，$u_{i}$ 和所有解释变量 $(x_{it},z_{i})$ 均不相关。
- 若假设成立，则 FE 和 RE 都一致，**但是 RE 比 FE 更有效率**
	- **效率**：**方差更小**
	- RE 引入的 $u_{i}$ 的均值 $E[u_{i}]$ 实际就是 FE 中的固定效应 $\bar{u}_{i}$
- **若假设不成立**，则 **FE 一致**，**RE 不一致**。

**不有效性**关注的是估计量的**精确性**（最小方差）。

- **核心问题：** 混合 OLS 忽略了复合扰动项中的 **序列相关性**。
    - 由于 $\epsilon_{it}^*$ 中包含了不随时间变化的 $u_i$，对于同一个体 $i$ 在不同时间 $t_1$ 和 $t_2$ 的扰动项，它们是相关的：
        $$\text{Cov}(\epsilon_{it_1}^*, \epsilon_{it_2}^*) = \text{Var}(u_i) \neq 0$$
    - 混合 OLS **错误地假设**所有观测值的扰动项是独立同分布的（即 $\text{Cov}(\epsilon_{it_1}^*, \epsilon_{it_2}^*) = 0$）。

**豪斯曼检验**（Hausman Test），又称**豪斯曼规范检验**（Hausman Specification Test），是计量经济学中用于选择**面板数据模型**（Panel Data Model）规范的一个关键统计检验。

它的核心作用是帮助研究者在**固定效应模型（FE）**和**随机效应模型（RE）**之间做出选择。

---

#### 1. 检验的核心目的

**目的：** 判断模型的个体特异性效应 $\boldsymbol{u_i}$ 是否与解释变量 $\boldsymbol{x_{it}}$ 相关。

- **固定效应（FE）模型：** **允许** $u_i$ 与 $x_{it}$ 相关（即存在内生性）。
    
- **随机效应（RE）模型：** **假设** $u_i$ 与 $x_{it}$ 不相关（即不存在内生性）。
    

豪斯曼检验通过比较这两种模型估计出的系数是否存在**显著的系统性差异**来做出选择。

#### 2. 检验的原理与假设

豪斯曼检验基于一个基本原理：如果随机效应的假设成立（即 $\text{Cov}(u_i, x_{it}) = 0$），那么 FE 和 RE 两种方法估计出的系数应该是相似的。

##### **原假设 ($H_0$) 与备择假设 ($H_1$)**

|**假设**|**描述**|**估计量性质**|**最终选择**|
|---|---|---|---|
|**原假设 ($H_0$)**|**个体效应 $\boldsymbol{u_i}$ 与 $\boldsymbol{x_{it}}$ 不相关。**|FE 和 RE 都是**一致**的，但 RE **更有效率**（方差更小）。|**选择随机效应 (RE)**|
|**备择假设 ($H_1$)**|**个体效应 $\boldsymbol{u_i}$ 与 $\boldsymbol{x_{it}}$ 相关。**|FE 是**一致**的；RE 是**不一致**的（存在偏差）。|**选择固定效应 (FE)**|

#### 3. 检验的统计量

豪斯曼检验统计量 $H$ 衡量的是 FE 估计量和 RE 估计量之间的差异程度。

$$H = (\hat{\boldsymbol{\beta}}_{FE} - \hat{\boldsymbol{\beta}}_{RE})' [\text{Var}(\hat{\boldsymbol{\beta}}_{FE}) - \text{Var}(\hat{\boldsymbol{\beta}}_{RE})]^{-1} (\hat{\boldsymbol{\beta}}_{FE} - \hat{\boldsymbol{\beta}}_{RE})$$

- $\hat{\boldsymbol{\beta}}_{FE}$：固定效应模型的系数估计量。
    
- $\hat{\boldsymbol{\beta}}_{RE}$：随机效应模型的系数估计量。
    
- 统计量 $H$ 渐近服从自由度为 $k$（解释变量数量）的**卡方分布**($\chi^2$)。
$$\boldsymbol{(\hat{\boldsymbol{\beta}}_{FE} - \hat{\boldsymbol{\beta}}_{RE})'} \underbrace{[\text{Var}(\hat{\boldsymbol{\beta}}_{FE}) - \text{Var}(\hat{\boldsymbol{\beta}}_{RE})]^{-1}}_{\text{中间的 } k \times k \text{ 矩阵}} \boldsymbol{(\hat{\boldsymbol{\beta}}_{FE} - \hat{\boldsymbol{\beta}}_{RE})}$$
	- **得到一个标量**。
- $'$ **代表一个转置**。
- 相当于用 `Var` 标准化后的误差**距离**

#### 4. 结论解读

|**检验结果**|**统计学意义**|**最终决策**|
|---|---|---|
|**$p \text{ 值} > 0.05$**|**不拒绝 $H_0$**。系数无显著差异。|接受 RE 模型的假设，**选择随机效应模型**（因为它更有效率）。|
|**$p \text{ 值} \le 0.05$**|**拒绝 $H_0$**。系数有显著差异。|接受 FE 模型的假设，**选择固定效应模型**（因为 RE 估计量不一致）。|

#### 总结

豪斯曼检验是面板数据分析中**模型规范选择的权威标准**。它通过判断**内生性**（即个体效应是否与解释变量相关）来指导模型的选择：
- **RE 更高效，但易有偏差。**
- **FE 永远一致，但效率较低。**
如果**豪斯曼检验**显著，意味着随机效应的假设被严重违反，你必须选择 FE 模型来保证估计的**一致性**






# 内生性
## 再回头看：内生性检验

内生性问题（Endogeneity Problem），也称为内生性偏差（Endogeneity Bias），是计量经济学和回归分析中一个**非常严重的核心问题**。它指的是当模型中的**解释变量（自变量）** 与 **误差项（扰动项）** 之间存在**相关性**时所引发的统计问题。

**没有内生性**，没人会觉得你识别了因果关系。
- 因为你带伞了，所以下雨了。
## 双向因果的例子 (内生性 )

供需均衡模型
用 **供需均衡模型（Supply and Demand Equilibrium Model）**来解释**双向因果关系（Simultaneity）**，这是导致计量经济学中**内生性问题**的一个经典来源。

---

### 双向因果关系（Simultaneity）的定义

**双向因果关系**是指模型中的两个或多个变量**同时互为因果**，它们在同一时间被决定，并且相互影响。

在标准回归中，我们通常假设：

$$X \rightarrow Y$$

（$X$ 影响 $Y$）

而在双向因果关系中，我们面临的是：

$$X \leftrightarrow Y$$

（$X$ 影响 $Y$，**同时** $Y$ 影响 $X$）

### 供需均衡模型的例子

我们来考虑一个简单的市场模型，其核心变量是**价格 ($P$)** 和**数量 ($Q$)**。

#### 1. 市场模型设定

市场的均衡价格 $P^*$ 和数量 $Q^*$ 是由以下两个结构方程同时决定的：

- 需求方程（Demand）： 消费者行为
    
    $$Q_D = \alpha_0 + \alpha_1 P + \alpha_2 Y + \epsilon_D$$
    
- 供给方程（Supply）： 生产者行为
    
    $$Q_S = \beta_0 + \beta_1 P + \beta_2 C + \epsilon_S$$
    
- 均衡条件：
    
    $$Q_D = Q_S = Q$$
    

其中：

- $P$：产品价格
- $Q$：交易数量
- $Y$：消费者收入（影响需求，外生变量）
- $C$：生产成本（影响供给，外生变量）
- $\epsilon_D, \epsilon_S$：随机冲击（如偏好变化、天气变化等）
    

#### 2. 双向因果关系的体现

**内生变量（Endogenous Variables）：$P$ 和 $Q$**

1. **价格 ($P$) 影响数量 ($Q$)：**
    
    - 根据需求定律（$\alpha_1 < 0$），价格越高，需求量越低。
        
    - 根据供给定律（$\beta_1 > 0$），价格越高，供给量越高。
        
2. **数量 ($Q$) 影响价格 ($P$)：**
    
    - $P$ 和 $Q$ 都是市场均衡的**结果**。市场的任何冲击（例如，某个因素导致数量 $Q$ 增加，如天气好转导致供给 $\epsilon_S$ 增加），都会**同时**导致均衡价格 $P$ 的变化。
        

因此，$P$ 和 $Q$ 是同时被决定的，它们之间存在**双向因果关系**。

### 3. 计量经济学中的内生性问题

现在，假设我们只对需求方程感兴趣，并尝试用 OLS 进行回归：

$$Q = \alpha_0 + \alpha_1 P + \alpha_2 Y + \boldsymbol{\epsilon}$$

（这里 $\epsilon$ 泛指包含 $\epsilon_D$ 和 $\epsilon_S$ 效应的最终残差）

#### OLS 失败的原因

在市场均衡中，价格 $P$ 受到**需求冲击 $\epsilon_D$** 和**供给冲击 $\epsilon_S$** 的影响。

1. **供给冲击（$\epsilon_S$）：** 如果天气好转（ $\epsilon_S$ 是正的），这会**增加供给**，导致**数量 $Q$ 增加**，并**同时导致价格 $P$ 下降**。
    
2. **需求冲击（$\epsilon_D$）：** 如果消费者突然偏好增加（ $\epsilon_D$ 是正的），这会**增加需求**，导致**数量 $Q$ 增加**，并**同时导致价格 $P$ 上升**。
    

由于 $P$ 受到这些随机冲击 $\epsilon$（它们包含在你的回归误差项中）的影响，因此：

$$\text{Cov}(P, \epsilon) \neq 0$$

- **结论：** 解释变量 $P$ 与误差项 $\epsilon$ 相关，即 $P$ 是**内生变量**。
    

如果直接使用 OLS 估计 $\alpha_1$，得到的系数 $\hat{\alpha}_1$ 将是**有偏且不一致**的，它无法准确估计需求曲线的真实斜率。

### 解决双向因果关系（内生性）

解决这种双向因果关系最常用的方法是**工具变量法（Instrumental Variables, IV）**，特别是**两阶段最小二乘法（2SLS）**。

- **工具变量的选择：** 在供需模型中，你需要找到一个只影响供给（$C$）但不影响需求（$Q_D$）的工具变量，或者一个只影响需求（$Y$）但不影响供给（$Q_S$）的工具变量，来**识别**（Identify）你感兴趣的那条曲线。

## 测量误差问题 (measurement error / error-in-variables)

假设真实模型是
$$
y=\alpha+\beta x^{*}+\varepsilon,\text{where},\beta\neq 0,Cov(x^{*},\varepsilon)=0
$$

but $x^{*}$ is unobservable, we use a proxy variable $x$, and we hold
$$
x=x^{*}+u,\text{where } Cov(x^{*},u)=0,\;Cov(u,\varepsilon)=0
$$

**于是利用代理变量的回归模型**，存在扰动项 $(\varepsilon-\beta u)$
(这是为了凑出：$y=\alpha+\beta(x^{*}+u)-\beta u+\varepsilon=\alpha+\beta x^{*}+(\varepsilon-\beta u)$)
此时：
$$
\begin{align}
 Cov(x,\varepsilon-\beta u)& =Cov(x^{*}+u,\varepsilon-\beta u) \\
 & =Cov(x^{*},\varepsilon)-\beta Cov(x^{*},u)+Cov(u,\varepsilon)-\beta Cov(u,u) \\
 & =-\beta Var(u)
\end{align}
$$
- 回忆 Gauss-Markov 假设 4 [[Gauss-Markov假设#Gauss-Markov 假设（五大条件）]]
可见，代理变量的测量误差会导致 **OLS** 估计有偏

## 虚拟变量构造双重差分 (DiD)
- **Difference-in-Differences, DiD**

**DiD**回归的构造
$$
Y_{it}=\beta_{0}+\beta_{1}Treat_{i}+\beta_{2}After_{t}+\beta_{3}(Treat_{i}\times After_{t})+\varphi X_{it}+\varepsilon_{it}
$$
- $After_{t}$ 对时间来分：某时间之前为 0，某时间之后为 1
- $Treat$ **是某个干预措施或政策 (Treatment)** 产生的净因果效应

- $After_{t}\times Treat_{i}$ 就是又在某时间后，又是要处理的。能获得 **差异**

$\boldsymbol{\beta_3}$ 之所以能代表净效应，是因为它通过以下两次差分排除了所有干扰项：

|**效应项**|**作用**|**对应系数**|
|---|---|---|
|**消除固有差异**|第一次差分（处理组的变化）减去（控制组的变化）时，消除了处理组和控制组**本身**的平均差异 $\beta_1$。| $\beta_1$ |
|**消除时间趋势**|第一次差分（干预后 - 干预前）时，消除了所有组别共同经历的时间趋势 $\beta_2$。| $\beta_2$ |

最终，$\boldsymbol{\beta_3}$ 隔离出的部分，就是政策实施后，处理组**额外**于控制组和时间趋势的变化，即**政策的净效果**。

OLS 会自动执行双重差分。

## 工具变量法
**基本思想**：**分离与扰动项相关的部分**
通常借助另外一个“**工具变量**”实现这种分离。

**有效工具变量 $Z$** **的两个条件**：
- **条件一**：相关性 (Relevance)
	- 工具变量和内生解释变量相关
$$
Cov(Z,X)\neq 0
$$
- **条件二**：与扰动项不相关 (外生性)(Exogeneity)
$$
Cov (Z,\varepsilon)=0
$$
当你有 $L \ge 2$ 个工具变量来处理 $M=1$ 个内生变量时，不再使用简单的 IV 比率公式，而是使用 **两阶段最小二乘法（2SLS）**：

#### 第一阶段（内生性清除）

用所有**外生变量**（包括所有有效的工具变量 $Z_1, Z_2, \dots$）对内生变量 $X$ 进行 OLS 回归：

$$X_i = \gamma_0 + \gamma_1 Z_{1i} + \gamma_2 Z_{2i} + \dots + \gamma_L Z_{Li} + \text{其他外生变量} + v_i$$

然后，得到 $X$ 的**拟合值** $\hat{X}$。这个 $\hat{X}$ 只包含了 $X$ 中与工具变量相关的、**外生的**那部分变异。

$$
\hat{X}_{i}=\hat{\gamma_{0}}+\hat{\gamma_{1}}Z_{1i}+\hat{\gamma}_{2}Z_{2i}+\dots+\hat{\gamma}_{L}Z_{Li}+其他外生变量+v
$$
- 估计结束别忘了加 `hat`
#### 第二阶段（估计系数）

用**拟合值 $\hat{X}$** 代替原始内生变量 $X$，对因变量 $Y$ 进行 OLS 回归：

$$Y_i = \beta_0 + \beta_1 \hat{X}_i + \text{其他外生变量} + \mu_i$$

第二阶段得到的系数 $\hat{\beta}_1$ 就是**一致**的 IV/2SLS 估计量。

### 思考讨论 ：
- **一个内生变量，能否有两个或以上工具变量？**
	- 可以，而且更准确。
- **两个内生变量，可否共用一个工具变量？**
	- 不可以。导致严重的共线性。
	- 缺乏独立的信息将 $Y$ 变动归因于 $X_{1}$ 还是 $X_{2}$
	- 除非这两个内生变量本身即是线性的（那为什么不合并？）
- **若工具变量和内生变量弱相关会如何？**

# 从线性设定到非线性设定的拓展

## 线性形式的误设
拉姆齐的设定误差检验（RESET）
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251118083158.png)


## 非线性回归和神经网络模型
看看回归模型的定义：

已知 $X$ 时，$Y$ 的条件期望是
$$
E[Y|X]=f(X)
$$
若 $f(X)=\alpha+\beta X$, 则为线性回归模型。然而，$f(X)$ 可以是任意非线性模型。

如果是模拟神经元的，可以构造出神经网络模型这个特殊的非线性回归模型

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251118083449.png)

### 多层感知机 (MLP, Multilayer Perceptron)
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251118083739.png)


## MLP 应用示例

- **神经网络的深度学习**：**自动编码器**（AutoEncoder）
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251118084713.png)

- 编码：$\boldsymbol{z=}f(\boldsymbol{W}^{(1)}\boldsymbol{x}+\boldsymbol{b}^{(1)})$
- 解码：$\boldsymbol{x'=}f(\boldsymbol{W}^{(2)}\boldsymbol{z}+\boldsymbol{b}^{(2)})$

**目标函数**：
$$
L=\sum_{n=1}^{N}||x_{( n)}-x'_{n}||^{2}+\eta \rho(Z)+\lambda||W||^{2}
$$

| **组成部分**                           | **表达式**          | **作用和含义**                                                                                                           |
| ---------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| **1. 重构误差** (Reconstruction Error) | $\sum_{n=1}^{N}$ |                                                                                                                     |
| **2. 稀疏性惩罚** (Sparsity Penalty)    | $\eta \rho(Z)$   | 这一项用于引入**稀疏性约束**。它旨在限制隐藏层**编码 $\boldsymbol{Z}$** 的神经元同时被激活的数量，迫使模型学习更具区分度的局部特征。其中， $\boldsymbol{\eta}$ 是稀疏惩罚项的权重参数。 |
| **3. 权重衰减** (Weight Decay) / 正则化项  | $\lambda$        |                                                                                                                     |



![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251202080857.png)
- 卷积 G
	- **是一种滤波**

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251202090659.png)


##  [[FE - 研究pre]]


# 自回归条件异方差 (ARCH) 模型
Autoregressive Conditional Heteroskedasticity Model
经典的 ARMA 模型描述了平稳序列的一阶矩过程
- ARCH **模型描述了平稳序列的二阶矩过程**
- ARCH 模型的主要动机是捕捉金融资产收益率波动构成中的“聚集性”特征


### 完整的 ARCH 模型设定
应该包括均值方程和方差方程

$$
y_{t}=\mu_{t}+u_{t}
$$
where $\mu_{t}$  is conditional expectation of $y_t$, and $u_{t}$ is disturbance item
- 均值方程
	- **移动平均**对 $u_{t}$. 所谓移动平均是指用 $\theta$ 对之前两期加权
$$
\mu_{t}=\phi_{1}y_{t-1}+\phi_{2}y_{t-2}+\theta_{1}u_{t-1}+\theta_{2}u_{t-2}
$$
	- 关于 $\mu_{t}$ 的方程就是均值方程

- **方差方程**
	- 扰动项，CLRM 设定 $u_{t}\sim_{iid}N(0,\sigma^{2})$, 即 $u_{t}$ 没有异方差性
	- **但是**
	- 创新点。
	- $\mathrm{ARCH}(q)$ 模型认为 $\sigma_{t}^{2}$ 和过去的扰动项 $u_{t}^{2}$ 有关
$$
\sigma_{t}^{2}=\alpha_{0}+\alpha_{1} u_{t-1}^{2}+\alpha_{2}u_{t-2}^{2}+\dots+\alpha_{q}u_{t-q}^{2}
$$
	- 系数满足非负性
		- $\forall i=0,1,\dots,q,\alpha_{i}\geq {0}$



