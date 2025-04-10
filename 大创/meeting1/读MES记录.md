
# 一、论文背景与意义

1. **系统性风险的重要性**：概述金融危机（如2007-2009年）中系统性风险对经济的影响。
2. **现有风险管理不足**：个体风险管理方法（如VaR）如何无法充分应对系统性风险。
3. **研究目标**：开发衡量和管理系统性风险的经济模型和方法。

摘要翻译：我们提出了一个**系统性风险的经济模型**，其中假设金融部门整体的资本不足会损害实际经济，从而导致系统性风险的外部性。每个金融机构对系统性风险的贡献可以通过其系统性预期缺口（Systemic Expected Shortfall, SES）来衡量，即在整个系统资本不足时，该机构资本不足的倾向。SES随着机构杠杆率的增加和边际预期缺口（MES，即系统损失分布尾部的损失）的增大而上升。我们通过实证分析展示了SES的各组成部分在预测2007-2009年金融危机中的系统性风险方面的能力。



---

# 二 、理论框架

1. **系统性风险的定义**：
    - 系统预期缺口 (SES)：在系统性事件发生时金融机构的资本短缺程度。
    - 边际预期缺口 (MES)：个体机构在系统尾部事件中的损失。
2. **模型基础**：
    - SES如何与金融机构的杠杆率和MES关联。
    - 用于系统性风险税的最优征税方案。

## 定义 
### SES 
与**一个银行在未来的系统性金融损失事件中，一个银行损失的期望值相等。**
SES, the systemic-risk component, is equal to the expected amount a bank is undercapitalized in a future systemic event in which the overall financial system is undercapitalized.

Financial firm’s marginal expected shortfall, **MES** (i.e., its losses in the tail of the aggregate sector' s loss distribution), and to its leverage.
*Measuring systemic-risk*论证了 SES 是 Measurable 的，且与 Marginal expect shortal, **MES**相关

### $ES$ - (Expected Shortfall)
$ES$ is the expected loss conditional on the loss being greater than the $VaR$:
$$
ES_{\alpha}=-E[R|R\leq-VaR_{\alpha}] 
$$
是在 $R\leq-VaR_{\alpha}$ 条件下的，对损失 $R$ 求的条件期望. 这表达当损失超过 $VaR_{\alpha}$ 时候的损失平均值。

在研究 [[#MES]] 的时候我们通常关注 ES 更多，而非 VaR。因为 VaR 只关注到 1%~5%的 $\alpha$，没有达到这个阈的风险可能被忽视(就可能造成严重的微小 Risk 累积起来造成的严重后果!)，但是 $ES_{\alpha}$ 可以关注到所有损失值，即使没有达到这个风险阈 $\alpha$。因为只要 $R\leq-VaR_{\alpha}$ 就会被关注到并计算进期望。

对于银行决策，整体性的 $ES_{\alpha}$ 并不能提供足够的信息。我们把它拆分成不同的部分 (trading desks or individual group) 来研究

将 $R$ 拆分为 $\sum_{i}y_{i}r_{i}$, where $y_i$ is the **weight** of group $i$ in the total portfolio (投资组合)
那么
$$
ES_{\alpha}=-\sum\limits_{i}y_{i}E[r_{i}|R\leq-VaR_{\alpha}] 
$$
对 $y_i$ 求偏导
$$
\frac{\partial ES_{\alpha}}{\partial y_{i}}=-E[r_{i}|R\leq-VaR_{\alpha}] \equiv MES^i_{\alpha}
$$
这就是**MES** 的定义

### MES

$$
\frac{\partial ES_{\alpha}}{\partial y_{i}}=-E[r_{i}|R\leq-VaR_{\alpha}] \equiv MES^i_{\alpha}
$$
$MES_{i}$ 表达了 group $i$ 的边际期望损失 (marginal expected shortfall)


一部分投资组合的风险可以用 MES 衡量，这启发我们一个银行在金融系统对风险的贡献 (distribution) 也可以用 MES 衡量。

## 文献回顾
作者进行了文献回顾
> Turning to the literature, one strand of recent papers on systemic risk takes a structural approach using contingent claims analysis of the financial institutions’ assets

Adrian and Brunnermeier (Forthcoming)
Measure the financial sector’s VaR given that a bank has had a VaR loss, which they denote **CoVaR**
有一部分采用了结构化方法，通过对金融机构资产的或有索偿分析来研究系统性风险（如 Lehar 2005；Gray、Merton 和 Bodie 2008；Gray 和 Jobst 即将发表）。然而，由于需要对金融机构的负债结构做出较强的假设，这种方法在实际应用中存在复杂性。作为替代，一些研究者使用市场数据推导出系统性风险的简化形式指标。
- **MES "bridge the gap"**  between the sructural and reduced-form approaches by considering a **simple economic model** that gives rise to a measure of systemic risk contribution that depends on observable data and statistical techniques that are related to those in the reduced-form approches and easily applicable by regulators

- 可自然地随公司规模进行调整，与许多简化形式方法不同。（特点、亮点）

- 和 CoVaR 有着很大的不同
	- SES MES examine a financial firm' s stress conditional on systemic stress
	- CoVaR examines the system' s stress conditional on an individual firm' s stress






---

# 三、实证分析

1. **金融危机中的风险预测能力**：
    - 2007-2009年金融危机中，SES与实际风险事件的相关性。
    - ![[Pasted image 20250226181451.png]]
    - [V-Lab: Systemic Risk Analysis Summary](https://vlab.stern.nyu.edu/srisk) 本网站可以查询。
    - MES和杠杆率在预测系统性风险中的作用。
2. **与监管实践对比**：
    - SES与2009年联储压力测试结果（SCAP）的比较。
    - 使用市场数据计算的SES与传统方法（如Beta）的优劣。

---

# 四、政策启示与实践应用

1. **系统性税的设计**：基于SES的风险税如何激励银行减少系统性风险。
2. **监管工具建议**：如何将SES和MES应用于未来的压力测试和资本规划。

在这部分作者引入了一些模型来为后文说明问题做准备。

## 银行激励（Bank Incentives）
### 建立模型
- The economy has $N$ financial firms, which we denote as banks, indexed by $i=1,\dots ,N$
- Each bank i chooses how much $x_{j}^{i}$ to invest in each of the available assets $j=1,\dots ,J$

总收入资本 (acquiring total assets) $a^{i}$
$$
a^{i}=\sum_{j=1}^{J}x_{j}^{i}
$$
这些投资可以分为：债务 (debt)和股权 (equity)
$$
w_{0}^{i}+b^{i}=a^{i}\tag{总预算约束公式}
$$
It means asset $a^{i}$ must equal the sum of **equity** $w_{0}^{i}$ and the  **debt** $b^{i}$

- 通常，银行 $i$ 的所有者有一 初始禀赋 (initial endowment) $\bar{w}_{0}^{i}$，其中有 $w_{0}^{i}$ 以股份资本的形式存在银行，其余分散到其他消费和使用活动中.



At time 1, asset $j$  pays off $r_{j}^{i}$  per dollar (r 是一个比率) for bank $i$, 
净利润 (net return)  为 $r_{j}^{i}-1$
Total market value of the bank assets at time 1 is 
$$
y^{i}=\hat{y}^{i}-\phi^{i}\tag{市场价值公式}
$$
Where $\phi^{i}$ 是金融困境成本 (costs of financial distress)
$\hat{y}^{i}$ 是 pre-distress 困境前的收入
$$
\hat{y}^{i}=\sum_{j=1}^{J}r_{j}^{i}x_{j}^{i}
$$
$\phi^{i}$ 依赖于银行资本的市场价值，和债务的市场面值 (face value), 是二者的函数
$$
\phi^{i}=\Phi(\hat{y}^{i},f^{i})
$$
- 银行有特殊的一点：他们部分的负债有政府担保. 他们的金融困境会造成（impose）系统性金融外部性（后面讨论政府福利）

$$
b_i=\alpha_{i}f_{i}+(1-\alpha _{i})\mathbb{E}\left[ \min(f_{i},y_{i})\right] \tag{债务定价公式}
$$
含义：债务 $b_{i}$ 的价值等于担保部分和非担保部分的综合
$\alpha_{i}$ is a **fraction**: 代表显性或隐性被征服担保的债务

这种取最小的设定，是为了保证债权人可以收支平衡 (break even)

银行的净价值 (net worth of the bank)，$w_{i}$，at time 1 is:
$$
w_{1}^{i}=\hat{y}^{i}-\phi^{i}-f^{i}
$$









---

# 五、讨论与展望

1. **模型的优势与局限**：
    - 简化模型如何填补理论与实践之间的鸿沟。
    - 对模型假设（如尾部分布）的讨论。
2. **未来研究方向**：扩展到国际金融机构、跨市场风险等。

---

# 解读材料

1. **关键概念图解**：
    - SES和MES的计算逻辑。
    - 系统性税的分配机制。
2. **数据支持**：
    - 表格：不同金融机构的SES、MES、杠杆率与压力测试结果。
    - 图示：SES与SCAP结果的相关性。
3. **模型公式与应用**：
    - 系统性风险公式推导的关键步骤。
    - 案例分析：如贝尔斯登和雷曼兄弟的SES排名及后果。


绘图
```mermaid
flowchart
A(optimal taxation policy) <--() B(each bank's expected capital shorfall)
```


- LVG:  Leverage
	- measureed as quasi-market value of assets divided by market value of equity.(资产的准市场价值除以股权的市场价值来衡量。)