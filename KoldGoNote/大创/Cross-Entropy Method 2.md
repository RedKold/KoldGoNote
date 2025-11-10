# Cross-Entropy (CE) Method for Importance Sampling in MES Estimation

## 1. Problem Definition

We aim to estimate the **Marginal Expected Shortfall (MES)**:

$$
\text{MES} = -\mathbb{E}[r_i \mid r_m \leq -\text{VaR}_\alpha]
$$

More generally, we consider the expectation of a function under some distribution $p(x)$:

$$
\theta^* = \mathbb{E}_p[h(X)] = \int h(x)p(x)\,dx
$$

However, when $h(x)$ depends on **rare events** (e.g. $r_m \leq -\text{VaR}$), direct Monte Carlo estimation is inefficient because most samples contribute nothing.

---

## 2. Importance Sampling (IS)

We rewrite the expectation under a proposal distribution $q_\theta(x)$:

$$
\mathbb{E}_p[h(X)] = \int h(x)\frac{p(x)}{q_\theta(x)} q_\theta(x)\,dx = \mathbb{E}_{q_\theta}[h(X) w_\theta(X)]
$$

where the **importance weight** is

$$
w_\theta(x) = \frac{p(x)}{q_\theta(x)}.
$$

A good choice of $q_\theta$ increases the probability of sampling from the rare event region, reducing estimator variance.

---

## 3. Cross-Entropy (CE) Principle

The CE method views the search for the *optimal proposal* as an **optimization problem**.

Theoretically, the *optimal importance sampling distribution* is:

$$
q^*(x) = \frac{h(x)p(x)}{\mathbb{E}_p[h(X)]}.
$$

Since $\mathbb{E}_p[h(X)]$ is unknown, we approximate $q^*$ by minimizing the Kullback-Leibler divergence:

$$
\theta^* = \arg\min_\theta D_{\text{KL}}(q^* \,\|\, q_\theta).
$$

Expanding the KL divergence:

$$
D_{\text{KL}}(q^* \,\|\, q_\theta)
= \int q^*(x) \log\frac{q^*(x)}{q_\theta(x)}\,dx
= \text{const} - \int q^*(x)\log q_\theta(x)\,dx.
$$

Since the first term does not depend on $\theta$, we have:

$$
\theta^* = \arg\max_\theta \mathbb{E}_{q^*}[\log q_\theta(X)].
$$

Substituting $q^*(x) \propto h(x)p(x)$, the objective becomes:

$$
\theta^* = \arg\max_\theta \int h(x)p(x)\log q_\theta(x)\,dx.
$$

Using samples from $p(x)$:

$$
\theta^* \approx \arg\max_\theta \sum_{i=1}^N h(x_i)\log q_\theta(x_i).
$$

This corresponds to a **weighted Maximum Likelihood Estimation (MLE)**, where samples are weighted by their importance $h(x_i)$.

---

## 4. CE + IS Algorithm

### Step 1. Initialization
Start with an initial proposal distribution $q_{\theta_0}(x)$  
(e.g. a Gaussian $\mathcal{N}(\mu_0, \Sigma_0)$).

### Step 2. Sampling
Draw samples $x_1, \dots, x_N$ from $q_{\theta_t}(x)$.

### Step 3. Compute Weights
Evaluate importance weights:

$$
w_i = \frac{p(x_i)}{q_{\theta_t}(x_i)}.
$$

### Step 4. Compute Indicator or Reward Function
Define:

$$
I_i = \mathbf{1}_{\{x_i \in \mathcal{A}\}} \quad \text{or} \quad h(x_i)
$$

to emphasize samples contributing to the tail event.

### Step 5. Update Parameters (CE / Weighted MLE)
Update the proposal parameters by maximizing:

$$
\theta_{t+1} = \arg\max_\theta \sum_i I_i w_i \log q_\theta(x_i).
$$

Optionally, use **elite samples** (top $\rho\%$ of samples ranked by $h(x_i)$) for stability.

### Step 6. Iterate
Repeat Steps 2–5 until convergence.

### Step 7. Final Estimation
Once $q_{\theta^*}$ is obtained, estimate MES using importance sampling:

$$
\widehat{\text{MES}} = -\frac{\sum_i r_i \, \mathbf{1}(r_m \leq -VaR) \, w_{\theta^*}(x_i)}
{\sum_i \mathbf{1}(r_m \leq -VaR) \, w_{\theta^*}(x_i)}.
$$

---

## 5. Advantages

| Method | Description |
|--------|--------------|
| Standard IS | Requires manual design of proposal distribution |
| CE + IS | Learns optimal proposal automatically via optimization |
| Exponential Twisting | A special parametric case (Gaussian shift) |
| CE Method | A general framework applicable to Gaussian, t, GMM, etc. |

---

## 6. Application to MES Estimation

- $p(x)$: Original joint distribution of $(r_m, r_i)$ (can be Gaussian, t, or GMM)  
- $q_\theta(x)$: Proposal distribution (updated iteratively via CE)  
- $h(x)$: Tail-event weighted function, e.g. $\mathbf{1}(r_m \leq -VaR) \times r_i$  
- $\theta$: Parameters of the proposal distribution (mean, covariance, degrees of freedom, etc.)  
- Objective: Minimize variance and improve tail-sample efficiency for stable MES estimation.

---

## 7. Summary

The Cross-Entropy (CE) method provides a **unified and flexible optimization framework** for designing efficient importance sampling distributions.  
By framing the rare-event estimation problem as a **weighted likelihood optimization**, CE enables automatic adaptation of the proposal distribution toward the most relevant regions — such as financial tail events in MES computation.


[[CE 实验]]
