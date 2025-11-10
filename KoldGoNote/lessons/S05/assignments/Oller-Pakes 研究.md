# Oller-Pakes 

- [阅读这篇文献](https://www.jstor.org/stable/pdf/2171831.pdf?refreqid=fastly-default%3A82264a194c95a4ec5cf24214e857312d&ab_segments=&initiator=&acceptTC=1)
- 标准的道格拉斯生产函数
$$
y_{it}=\beta_{0}+\beta_{a}a_{it}+\beta_{k}k_{it}+\beta_{l}l_{it}+\omega_{it}+\eta_{it}
$$
- 参数解释
	- $y_{it}$: the $\log$ of output (value add) from plant $i$ at time $t$
		- 研究语境：Telecommunications Equipment Industry
	- $a_{it}$: its age
	- $k_{it}$: the $\log$ of its capital input
	- $l_{it}$ : the $\log$ of its labor input
	- $\omega_{it}$: its productivity
	- $\eta_{it}$: 误差项构成。测量误差、不可预见的生产率冲击。

## 问题
问题：
- **同时性偏差**：为什么用 OLS 来估计生产函数会产生偏差？
	- 将生产率 ($w_{it}$)视为外生变量会导致估计偏差。
	- **投入的内生性**（endogeneity of input demands）
		- 企业在决定投入时，会根据自己对生产率 $\omega_{it}$ 的预期调整。
		- 如果生产率在时间上有 **序列相关性** ，那么当期投入就会和当期生产率 $w_{it}$ 正相关。
	- OLS 偏差
		- 没有考虑未观测的生产率。
- **样本选择性偏差** （self-selection bias）
	- **企业退出市场**引起的偏差。
	- 企业是否退出，取决于它的**生产率** $\omega_{it}$、资本 $k_{it}$ 和年龄 $a_{it}$ 等状态变量。低生产率或资本较小的企业更可能退出市场。
	- 如果只用 **存活企业的数据** 来估计生产函数，那么样本已经被“筛选”过了


## 如何解决问题？

### 行为框架

- accumulation equations for capital and age 资本存量和年龄随时间演化的确定性方程
$$
k_{t+1}=(1-\delta)k_{t}+i_{t}
$$
	- 资本每期折旧 $\delta k_{t}$ 后，加上当期投资 $i_{t}$ 得到下一期资本
	- 年龄自然增长 `1`

- 考虑企业的退出
$$
\chi_t=
\begin{cases}
1 & \mathrm{~if~}\omega_t\geq\underline{\omega}_t(a_t,k_t), \\
0 & \text{ otherwise,} & 
\end{cases}
$$
	- 该指示函数，刻画了企业退出市场的行为。
	- 如果 $\omega_{t}\geq \underline{\omega}_{t}(a_{t},k_{t})$，即选择的状态变量表明，继续经营有利可图，企业会留在市场。
		- $\underline{w}_{t}(a_{t},k_{t})$ 是退出阈值生产率
			- **企业在当前状态下继续经营的最低生产率水平**
			- 依赖于资本 $k_{t}$ $a_{t}$

### 构造条件期望来考虑偏差

$$
E[\omega_{t}|a_{t},k_{t},\omega_{t-1},\chi_{t}=1]
$$
- 该估计，是考虑了**序列相关性**（引入了 $w_{t-1}$）
	- **思考**：为什么只引入 $\omega_{t-1}$ 是可能的？
		- 假设生产率服从一阶马尔可夫过程（first-order Markov）
- **缓解了样本选择性偏差**
	- 构造了退出阈值。捕捉退出导致的样本选择性。
	- 条件期望表达了 **在已存活企业中，生产率如何系统地依赖资本、年龄和上一期生产率**
	- 换句话说，它**量化了退出行为对观测样本中生产率分布的筛选作用**。


- **控制函数法（Control Function Approach, CF）**
- 在合适的假设（$i_{t}$ （投资）关于 $\omega_{t}$ 严格递增）下，我们可以求逆。
$$
\omega_{t}=h_{t}(i_{t},a_{t},k_{t})
$$
- 我们用 $h_{t}(i_{t},a_{t},k_{t})$ 代替未观测的 $w_{t}$，代入最初的道格拉斯生产函数

就得到

$$
y_{it}=\beta_{l}l_{it}+\phi_{t}(i_{it},a_{it},k_{it})+\eta_{it}
$$
- where $\phi_t(i_{it},a_{it},k_{it})=\beta_{0}+\beta_{a}a_{it}+\beta_{k}k_{it}+h_{t}(i_{it},a_{it},k_{it})$

# Levinsohn–Petrin

**对 Oller-Pakes 的方法进行了改进**

- Oller-Pakes 使用 $i_{it}$ 投资作为代理变量，但这只对非零投资的工厂有效
- 在样本中，研究发现有超过一半的工厂报告零投资，导致大量样本不可用
- **使用中间投入**作为代理变量，而不是**投资**。可以避免这个问题。
	- 至少在作者使用的数据中，公司总是有正的中间投入（原材料、电力等等）

$$
\iota_{t}=\iota_{t}(\omega_{t},k_{t})
$$
引入了一个 $\iota$，作为 **中间投入**(intermediate input, perhaps materials or energy)

则
$$
y_{t}=\beta_{0}+\beta_{l}l_{t}+\beta_{k}k_{t}+\beta_{\iota}\iota_{t}+\omega_{t}+\eta_{t}
$$
在合适的假设下，同样的，可以反演出 $w_{t}$ 的表达式 $w_{t}=w_{t}(\iota_{t},k_{t})$

其余的方法大同小异。


# 总结

OP 法和 LP 法都一定程度下围绕传统道格拉斯生产函数的内生性问题和样本选择性问题进行了改善。其中 OP 法通过考虑企业退出和考虑 $\omega_{it}$ 的时间序列依赖性进行了改进，而 LP 法针对 OP 法使用的代理变量 **投资** 做出了优化——尝试使用中间投入来作为变量。

但是二者，没有对道格拉斯生产函数忽略技术进步等因素做出很**明显的改进**

---
在实证研究中，进行估计步骤大概如下。
1. 数据准备
	- 收集企业工厂层面的面板数据。
2. 选择代理变量
	1. OP 使用 $I_{it}$ 投资
	2. LR 使用 $M_{it}$ 中间投入
3. 处理企业退出问题
	1. 通过构造条件期望
4. 估计未观测生产率 $\omega_{it}$
5. 代回经典道格拉斯生产函数，进行 OLS 回归。