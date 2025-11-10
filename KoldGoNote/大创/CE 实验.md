## 直接使用 CE 筛样本



## 使用 CE -策略为最小方差
`
![](https://kold.oss-cn-shanghai.aliyuncs.com/Experiment%20-%20min%20the%20variance.png)


- 该方法确实方差较小


- 一个别的对比

![RMSE - ESS -Varicance - Bias.png](https://kold.oss-cn-shanghai.aliyuncs.com/RMSE%20-%20ESS%20-Varicance%20-%20Bias.png)

- 最小方差确实方差极小（比普通 IS 还少一个数量级），但是带来较大的 `Bias`


### 实验问题：

如果用 IS 方法的大样本作为 `true_mes`，则估计偏差较大，导致 `RMSE` 几乎不收敛。
使用最小方差作为优化目标，`RMSE` 会变小。

- 既然选择的原始分布一样，理应和 `IS` 估计真值一样，但是结果偏差较大，可以考虑更换优化目标。



## 用 `KL散度` 来优化

![ce_exponential_tilting_KL.png|00](https://kold.oss-cn-shanghai.aliyuncs.com/ce_exponential_tilting_KL.png)

在 `exponential tilting` 的基础上做 IS，对比 Crude Monte Carlo，取得了比较好的效果。
- 原始分布：
- 提议分布：指数扭曲，`theta` 由 `CE` 方法优化得到

![CE optimized Exponential Tilting Convergence.png|00](https://kold.oss-cn-shanghai.aliyuncs.com/CE%20optimized%20Exponential%20Tilting%20Convergence.png)



![image.png|700](https://kold.oss-cn-shanghai.aliyuncs.com/20251025202641.png)


## Assignment
- **写一个 Note**给老师讲讲细节

- 实验上：
	- 画更细化的图-为什么 KL 散度下降这么快？
- 




## Note

## 实验流程概览

- **目标**：通过交叉熵（CE）方法优化指数扭曲（Exponential Tilting）的提议分布，用于估计市场下行条件下的边际期望损失（MES），并与原始蒙特卡罗及未调参的 IS 方法比较效率与稳定性。

### 数据准备

- **输入数据**：标普 500 指数 `data/GSPC_adj_close.csv` 与 JPM 股票 `data/individual_equities/2007-01-01_2009-12-31/JPM_adj_close.csv`。
- **处理流程**：`prepare.prepare_data_for_mes` 读取并对齐两组收益率，输出二元高斯分布的均值 `mu`、协方差 `Sigma` 以及市场收益的 `VaR`。
```python 163:172:ce_exponential_tilting.py
        data, mu, Sigma, VaR, _ = prepare.prepare_data_for_mes(
            asset_returns, market_returns, alpha=alpha
        )
```
- **基准估计**：`compute_naive_mes` 实现的原始蒙特卡罗为后续比较提供参考。
```python 434:488:ce_exponential_tilting.py
def crude_monte_carlo(asset_returns, market_returns, alpha=0.05, n_samples=10000, verbose=True):
    ...
    mes_estimate = compute_naive_mes(asset_returns, market_returns, alpha=alpha)
```

### 模型与分布设定

- **原始分布**：二维正态分布 `N(mu, Sigma)`，变量分别对应资产收益 `r_i` 与市场收益 `r_m`。
- **指数扭曲**：给定参数 `theta`，提议分布变为 `N(mu + Sigma * theta, Sigma)`。
```python 17:43:ce_exponential_tilting.py
def exponential_tilting_distribution(original_mu, original_Sigma, theta):
    ...
    tilted_mu = original_mu + np.dot(original_Sigma, theta)
    tilted_Sigma = original_Sigma.copy()
```
- **重要性权重**：`w(x) = f(x)/g(x)`，利用原始与扭曲分布的对数密度差计算。
```python 46:66:ce_exponential_tilting.py
def compute_importance_weights(samples, original_mu, original_Sigma, tilted_mu, tilted_Sigma):
    ...
    log_weights = log_f - log_g
    weights = np.exp(log_weights)
```

### CE 优化流程（KL 目标）

1. **初始化**：在 `ce_optimize_tilting_parameter` 中，以零均值、0.1 对角协方差构造 `theta` 的采样分布。
	- 我们指数扭曲的对象是 **多元高斯分布**
2. **采样 `theta`**：每轮从当前 `theta` 分布抽取 `n_samples` 个参数；对每个参数：
   - 计算倾斜后的分布并模拟样本；
   - 用软指示函数近似事件 `r_m ≤ -VaR`，获得 MES 估计；
   - 记录 KL 值、估计方差及表现分数（此处以最小化 KL 为目标）。
```python 188:234:ce_exponential_tilting.py
        for theta in theta_samples:
            tilted_mu, tilted_Sigma = exponential_tilting_distribution(mu, Sigma, theta)
            samples = tilted_dist.rvs(size=n_samples)
            weights = compute_importance_weights(samples, mu, Sigma, tilted_mu, tilted_Sigma)
            soft_indicator = 1 / (1 + np.exp(k_soft * (r_m + VaR)))
            weighted_mes = -r_i * soft_indicator * weights
            ...
            kl_div = compute_kl_divergence(theta, mu, Sigma, samples)
            performance_score = -kl_div
```
3. **精英更新**：挑选表现最佳的前 `elite_fraction` 样本，更新 `theta` 的均值和协方差，并记录历史指标（MES、方差、KL、RMSE）。
```python 242:313:ce_exponential_tilting.py
        elite_indices = np.argsort(performance_scores)[-n_elite:]
        elite_thetas = theta_samples[elite_indices]
        theta_mu = np.mean(elite_thetas, axis=0)
        theta_sigma = np.cov(elite_thetas.T) + np.eye(2) * 1e-6
        ...
        history['kl_divergence_history'].append(compute_kl_divergence(theta_mu, mu, Sigma, None))
```
4. **输出**：返回最优 `theta`、最终 MES 及历史记录，用于可视化 KL、RMSE、方差的收敛情况。


> [!Note] 这里的 KL 散度是如何计算的？
> `compute_kl_divergence` 返回原分布 `f = N(μ, Σ)` 与指数扭曲后的提议分布 `g_θ = N(μ + Σθ, Σ)` 的 KL 发散。因为两者协方差一致（都是 `Σ`），KL 发散有解析解，只取决于均值差：
> 
> $$ 
> D(f || g_\theta) = \tfrac{1}{2} (\mu - (\mu + Σθ))^\top Σ^{-1} (\mu - (\mu + Σθ))
> = \tfrac{1}{2} θ^\top Σ θ
> $$ 
> 
> - 协方差项抵消，KL 只与 `θ` 和 `Σ` 相关；
> - `θ` 越接近零（或 `Σ` 越小），KL 越小，表示扭曲后分布越贴近原分布；
> - 这是 CE 中的优化目标：最小化 `θ^T Σ θ` 等价于最小化 KL，从而找到最能兼顾原分布与目标事件的扭曲参数。
- a proof to the **analytical solution** of `KL` divergence.
![JPEG图像-4C5D-B66D-18-0.jpeg|400](https://kold.oss-cn-shanghai.aliyuncs.com/JPEG%E5%9B%BE%E5%83%8F-4C5D-B66D-18-0.jpeg)

```python:69
def compute_kl_divergence(theta, original_mu, original_Sigma, target_samples):
    """
    Compute KL divergence D(f||g_θ)
    
    For exponential tilting, KL divergence has analytical solution:
    D(f||g_θ) = 0.5 * θ^T * Σ * θ
    """
    kl_div = 0.5 * np.dot(theta, np.dot(original_Sigma, theta))
    return kl_div
```


### 重要性抽样评估

- **函数**：`exponential_tilting_is` 接受任意 `theta`，执行一次重要性抽样估计，输出 MES、有效样本量（ESS）及估计方差，便于与基线比较。
- **比较框架**：`compare_methods` 使用优化得到的 `theta`，与 `Crude_MC` 与 `theta = 0` 的 IS 进行多次重复试验，统计均值、方差、ESS、RMSE 与成功率，并绘制对比图。

### 可视化与实验发现

- **KL 收敛**：`ce_exponential_tilting_KL.png` 与 `CE optimized Exponential Tilting Convergence.png` 展示 KL 和 MES 随迭代下降的趋势，说明提议分布逐渐贴近目标分布。
- **RMSE/ESS/Bias 对比**：`RMSE - ESS -Varicance - Bias.png` 反映不同策略下的误差、效率与偏差权衡；最小方差策略方差极低，但偏差显著。
- **整体对比**：`CE optimized Exponential Tilting Convergence.png`、`20251025202641.png` 展示 KL 目标下的收敛轨迹及与 Crude Monte Carlo 的性能差异。

### 其他说明 

- 深入分析 KL 下降速度：输出更高分辨率的 KL vs. iteration 图，检查初始参数及权重集中情况；
- 丰富实验：在更多资产/时间区间上重复流程，验证稳健性，形成可阅读的实验报告供导师审阅。

此流程清晰地展现了从原始数据处理、提议分布优化到性能评估的完整路径，便于导师理解并提出针对性建议。



![image.png|900](https://kold.oss-cn-shanghai.aliyuncs.com/20251110131156.png)
- ` kl_history_tail_zoom.png`：一个放大了分辨率，**分别看前五次和后十次的例子**。
- 可见最后 10 次迭代，效果实际已经不明显了，在波动。


![ce_optimized_exponential_tilting_rmse_convergence.png|900](https://kold.oss-cn-shanghai.aliyuncs.com/ce_optimized_exponential_tilting_rmse_convergence.png)

- **我们的工作流程**：
	- 每次迭代，找到一个当前认为最好的 $\theta$，在其基础上做重要性采样，得到 `mes`
	- 由于迭代目标 `KL` 散度迅速收敛，我们之后的 `MES` 实际在一个范围内波动。


#### 代码层面解读计算步骤
`ce_optimize_tilting_parameter` 在每次迭代里为候选 `θ` 计算 MES，流程分三步：
1. **生成扭曲分布样本**  
   - 根据候选 `θ` 得到扭曲分布 `g_θ = N(μ + Σθ, Σ)` (`tilted_mu`, `tilted_Sigma`)。  
   - 从该分布模拟 `n_samples` 组 `(r_i, r_m)`。  
   ```python 197:205:ce_exponential_tilting.py
   tilted_mu, tilted_Sigma = exponential_tilting_distribution(mu, Sigma, theta)
   tilted_dist = multivariate_normal(mean=tilted_mu, cov=tilted_Sigma)
   samples = tilted_dist.rvs(size=n_samples)
   ```
1. **重要性权重与软指示器**  
   - 计算原分布 `f` 与扭曲分布 `g_θ` 的密度比 `w(x)=f(x)/g_θ(x)`；  
   - 使用软指示函数 `soft_indicator = 1 / (1 + exp(k_soft (r_m + VaR)))` 近似事件 `r_m ≤ -VaR`。  

   ```python 07:214:ce_exponential_tilting.py
   weights = compute_importance_weights(samples, mu, Sigma, tilted_mu, tilted_Sigma)
   soft_indicator = 1 / (1 + np.exp(k_soft * (r_m + VaR)))
   weighted_mes = -r_i * soft_indicator * weights
   weighted_indicator = soft_indicator * weights
   ```
1. **归一化求 MES 与方差**  
   - 只要软指示加权的权重和不为零，就以重要性加权平均计算 MES：
  $$ 
     \widehat{MES}_θ = \frac{\sum (-r_i)\, 1_{\text{soft}}(r_m)\, w (x)}{\sum 1_{\text{soft}}(r_m)\, w (x)}
 $$ 
   - 同时记录估计方差 `np.var(weighted_mes / (weighted_indicator + 1e-10))`。  
   ```python 16:303:ce_exponential_tilting.py
   if np.sum(weighted_indicator) > 0:
       mes_est = np.sum(weighted_mes) / np.sum(weighted_indicator)
       ...
       current_mes = np.sum(weighted_mes) / np.sum(weighted_indicator)
       current_var = np.var(weighted_mes / (weighted_indicator + 1e-10))
   ```
因此，每个候选 `θ` 都通过“从 `g_θ` 采样 → 用 `f/g_θ` 权重修正 → 只对下行尾部样本求加权平均”来得到 MES。CE 方法借此把 MES 表现最好（此处意思为 KL 最小）的 `θ` 纳入精英集，逐步逼近**最优扭曲参数。**
