Gauss-Markov 假设（有时称为经典线性回归模型假设 CLM 的子集）是保证 OLS（普通最小二乘法）估计量具有 **BLUE**（Best Linear Unbiased Estimator，最佳线性无偏估计量）性质的关键条件。

以下是五条主要的 Gauss-Markov 假设：

---

### Gauss-Markov 假设（五大条件）

|**编号**|**假设名称**|**符号表示**|**统计意义**|**保证的性质**|
|---|---|---|---|---|
|**1.**|**线性于参数**|$y_i = \mathbf{x}_i \boldsymbol{\beta} + \epsilon_i$|模型形式正确，且参数是线性的。|OLS 适用|
|**2.**|**随机抽样**|Data from i.i.d. sample|观测值来自随机且相互独立的样本。|统计推断基础|
|**3.**|**解释变量变异性**|$\mathbf{X}'\mathbf{X}$ 可逆|不存在**完全多重共线性**。|OLS 估计量可计算|
|**4.**|**零条件均值**|$E[\boldsymbol{\epsilon} \mid \mathbf{X}] = \mathbf{0}$|解释变量与误差项不相关（排除**内生性**）。|**无偏性 (Unbiasedness)**|
|**5.**|**球形扰动项**|$\text{Var}(\boldsymbol{\epsilon} \mid \mathbf{X}) = \sigma^2 \mathbf{I}$|**同方差** ($\text{Var}(\epsilon_i)=\sigma^2$) 且 **无自相关** ($\text{Cov}(\epsilon_i, \epsilon_j)=0$)。|**效率 (Best/Efficiency)**|

### Gauss-Markov 定理结论

如果以上 **1 到 5 条假设**全部成立，那么 OLS 估计量 $\hat{\boldsymbol{\beta}}$ 就是 **BLUE**（在所有线性无偏估计量中，它是方差最小的）。

### 第六条假设（用于统计推断）

虽然不影响 BLUE 性质，但为了在小样本下进行精确的 $t$ 检验和 $F$ 检验，通常还需要第六条假设：

| **编号** | **假设名称** | **符号表示**                                                                         | **意义**     |
| ------ | -------- | -------------------------------------------------------------------------------- | ---------- |
| **6.** | **正态性**  | $\boldsymbol{\epsilon} \mid \mathbf{X} \sim \mathcal{N}(0, \sigma^2 \mathbf{I})$ | 误差项服从正态分布。 |