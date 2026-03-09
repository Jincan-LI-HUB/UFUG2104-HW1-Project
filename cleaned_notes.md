我会在保留你们现有“每段代码的用法说明（不删）”的前提下，把 `notes_on_assignment_1.md` 重排成**更可检索、更少重复**、并补上你缺的 **Global conventions + Bugbook + Part i 的理论 4 行版 + Part f 公式修正**（你们原稿里 Part f 那个公式写成了逗号形式，数学上会误导）。

下面给你一份**“更详细版（越详细越好）”**的 `.md` 源码——直接整段复制保存即可（建议文件名：`notes_on_assignment_1_detailed.md`）。

````md
# Notes on Programming Assignment 1 (Detailed, Optimized) 📌

> Goal: Provide a *copy-paste runnable* + *math-linked* + *code-focused* set of notes for Programming Assignment 1.ipynb.
> This version is intentionally detailed and keeps **all code usage explanations**, while removing redundancy by:
> - Adding a **Global Conventions** section (single source of truth).
> - Consolidating reusable code into **Appendix: Reusable Code Snippets**.
> - Adding a **Bugbook** (common errors + fixes).
> - Fixing the **Part f total probability formula** (must be weighted sum, not commas).

---

## Table of Contents
- 0. Quick Start (a→i in ~10 minutes)
- 1. Global Conventions (One source of truth)
- 2. Section 1: One-dimensional modeling (Part a–f)
  - Part a: Normalized histogram
  - Part b: Empirical mean & std
  - Part c: Single Gaussian vs histogram
  - Part d: Histogram by species
  - Part e: Conditional Gaussian per species
  - Part f: Total probability mixture model vs single Gaussian
- 3. Section 2: Two-dimensional joint modeling (before Section 3)
  - Example: Scatter plot
  - Example: mean/cov/corr for joint variables
  - Example: MVN contour overlay (fitted joint Gaussian)
  - Example: Grouped scatter by Class
- 4. Part g–i: Species stats + classifier error curve
  - Part g: mean/cov/corr by species
  - Part h: answers (3 questions)
  - Part i: classifier D = 7W − L, error vs threshold T
- 5. Bugbook (Most common mistakes)
- Appendix A: Reusable Code Snippets (copy-paste toolbox)

---

# 0) Quick Start (a→i in ~10 minutes) ✅

If you want to run the notebook fast and still keep everything consistent:

1) Run Part a to create a normalized histogram of **petal length** on `[2,7]` with bin width `0.1`.
2) Run Part b to compute `m` and `sigma` (sample std, `ddof=1`).
3) Run Part c to overlay the Gaussian pdf and histogram.
4) Run Part d to show histograms by species.
5) Run Part e to fit **conditional** Gaussians for versicolor and virginica separately.
6) Run Part f to build **mixture** model `g_X(x)` via total probability and compare to Part c.
7) Run Section 2 examples to understand joint modeling: scatter, covariance/correlation, MVN contour.
8) Run Part g/h to get per-class mean/cov/corr.
9) Run Part i to plot error probability vs `T`, find `T*`, and compare to using one feature alone.

---

# 1) Global Conventions (One source of truth) 🧩

**These conventions MUST be consistent across parts**, otherwise comparisons become apples-to-oranges:

## 1.1 Plot ranges & grids
- **Histogram x-range:** $x ∈ [2, 7]$
- **Bin width:** $Δ = 0.1$
- **Bin edges:** use `np.arange(2.0, 7.0 + Δ + 1e-12, Δ)`  
  (the `1e-12` prevents floating-point edge loss)
- **Smooth curve grid:** `x_grid = np.linspace(2.0, 7.0, 600)`  
  (dense enough for smooth pdf curves)

## 1.2 Normalized histogram definition
Use `density=True` in `plt.hist`:
- `density=True` makes the histogram a **probability density estimate**:
  - bar height ≈ $count / (N * Δ)$
  - total **area** ≈ 1  
  (Important: not “sum of heights = 1”. It is “sum of areas = 1”.)

## 1.3 Sample vs population convention
- Use **sample** standard deviation (denominator `n-1`):
  - `Series.std(ddof=1)`  
- Use pandas `cov()` (also sample covariance by default, `n-1`)

## 1.4 DataFrame vs Series
- For one column, prefer `data["Petal length"]` (Series).
- `data[["Petal length"]]` is a DataFrame and may return Series results that cause broadcasting bugs later.

---

# 2) Section 1: One-dimensional modeling (Part a–f) 📌

---

## Part a: Normalized Histogram (bin=0.1, x∈[2,7]) ✅

### ✅ Code (paste into Part a code cell)
```python
# Part a: normalized histogram of petal length (all species), from 2 to 7, bin size 0.1

x = data["Petal length"].dropna()

bin_width = 0.1
# 用 arange 明确控制 bin 宽度；加一个很小的量避免浮点导致 7.0 没被包含为边界
bins = np.arange(2.0, 7.0 + bin_width + 1e-12, bin_width)

plt.figure(figsize=(7, 4))
plt.hist(x, bins=bins, density=True, edgecolor="black")
plt.xlim(2, 7)

plt.xlabel("Petal length")
plt.ylabel("Probability density (normalized histogram)")
plt.title("Normalized Histogram of Petal Length (All Species)")
plt.grid(alpha=0.3)
plt.show()
```

### Why this code (code usage explanation + math link)

* `dropna()` avoids NaNs breaking the histogram or silently affecting counts.
* `bins = np.arange(...)` forces exact bin width 0.1; the `+1e-12` avoids missing the endpoint due to float error.
* `density=True` converts “counts” to “density” so the histogram approximates a pdf:
  $$
  \text{height}_k \approx \frac{\text{count}_k}{N\Delta}
  $$
  and total area ≈ 1.

### Checklist (things to remember)

1. `density=True` → **area** = 1, not “sum of bar heights”.
2. Fix bin edges explicitly when a bin width is required.
3. Keep the same range `[2,7]` for all comparisons with pdf curves.

---

## Part b: Empirical mean m and std σ ✅

### ✅ Code (paste into Part b code cell)

```python
# Part b: empirical mean m and empirical standard deviation sigma of petal length (all species)

x = data["Petal length"].dropna()   # remove NaN if any

m = x.mean()
sigma = x.std(ddof=1)              # ddof=1 -> denominator (n-1), matches the formula in the prompt

print(f"Empirical mean m = {m:.6f}")
print(f"Empirical std  σ = {sigma:.6f}")
```

### Why this code (math link)

* Mean:
  $$
  m=\frac{1}{n}\sum_{i=1}^{n}r_i
  $$
* Sample std (assignment uses `n-1`):
  $$
  \sigma=\sqrt{\frac{1}{n-1}\sum_{i=1}^{n}(r_i-m)^2}
  $$
* `std(ddof=1)` matches denominator `(n-1)`.

### Checklist

* `ddof=1` is essential if the assignment states `(n-1)`.
* Prefer Series computations to avoid returning vectors/Series unexpectedly.

---

## Part c: Compare histogram with Gaussian pdf (single Gaussian) ✅

### Modeling assumption

Assume:
$$
X \sim \mathcal N(m,\sigma^2)
$$
where `m`, `sigma` come from Part b.

### ✅ Code (paste into Part c code cell)

**IMPORTANT:** force `m` and `sigma` to scalars to avoid broadcasting bugs.

```python
from scipy.stats import norm

# 1) Data
x = data["Petal length"].dropna()

# 2) Ensure scalar parameters (prevents (600,) vs (2,) broadcasting bug)
m = float(x.mean())
sigma = float(x.std(ddof=1))

# 3) Histogram settings (match Part a)
bin_width = 0.1
bins = np.arange(2.0, 7.0 + bin_width + 1e-12, bin_width)

# 4) Gaussian pdf on a grid
x_grid = np.linspace(2.0, 7.0, 600)
pdf = norm.pdf(x_grid, loc=m, scale=sigma)

# 5) Plot overlay
plt.figure(figsize=(7, 4))
plt.hist(x, bins=bins, density=True, edgecolor="black", alpha=0.6, label="Normalized histogram")
plt.plot(x_grid, pdf, linewidth=2, label="Gaussian pdf (m, σ from Part b)")
plt.xlim(2, 7)

plt.xlabel("Petal length")
plt.ylabel("Probability density")
plt.title("Petal Length: Empirical vs Single Gaussian Model")
plt.grid(alpha=0.3)
plt.legend()
plt.show()

print(f"m = {m:.6f}, sigma = {sigma:.6f}")
```

### Why this code (code usage explanation)

* `norm.pdf(x, loc, scale)` expects scalar `loc` and scalar `scale`.
* If `m` or `sigma` is a pandas Series (e.g., because you accidentally computed mean/std of multiple columns), you’ll get:

  * broadcasting error like shapes `(600,)` vs `(2,)`.
* `float(...)` hardens them into scalars.

### Part c Answer template (what to write)

* EN: The single Gaussian is **not** a great model for the full dataset because combining species can create multi-modality or asymmetry, while a Gaussian is unimodal and symmetric.
* ZH: 单一高斯对混合物种的数据往往不够好：总体分布可能多峰/偏斜，而高斯必然单峰对称。

---

## Part d: Normalized histogram for each species separately ✅

### Idea (stats link)

This is switching from marginal `f_X(x)` to conditional distributions by class: you visually compare
$$
f_{X|C}(x|c)
$$
for each species `c`.

### ✅ Code (paste into Part d code cell)

```python
# Part d: normalized histogram of petal length for each species (Class) separately

bin_width = 0.1
bins = np.arange(2.0, 7.0 + bin_width + 1e-12, bin_width)

groups = data.groupby("Class")

fig, axes = plt.subplots(1, len(groups), figsize=(14, 4), sharey=True)

for ax, (cls, g) in zip(axes, groups):
    g["Petal length"].dropna().plot.hist(
        bins=bins,
        density=True,          # normalized histogram
        edgecolor="black",
        alpha=0.7,
        ax=ax
    )
    ax.set_xlim(2, 7)
    ax.set_title(cls)
    ax.set_xlabel("Petal length")
    ax.grid(alpha=0.3)

axes[0].set_ylabel("Probability density")
fig.suptitle("Normalized Histogram of Petal Length by Species (Class)", y=1.02)
plt.tight_layout()
plt.show()
```

### Code usage explanation

* `groupby("Class")` splits rows into separate DataFrames by species.
* Using shared bins and shared y-axis makes the shape comparison fair.

### Checklist

* Always use `density=True`.
* Always reuse bins when comparing multiple histograms.

---

## Part e: Conditional Gaussian models for versicolor & virginica ✅

### Modeling assumption

For each species (s):
$$
X|C=s \sim \mathcal N(m_s,\sigma_s^2)
$$
Estimate `m_s`, `sigma_s` from that species’ sample.

### ✅ Code (paste into Part e code cell)

```python
from scipy.stats import norm

bin_width = 0.1
bins = np.arange(2.0, 7.0 + bin_width + 1e-12, bin_width)
x_grid = np.linspace(2.0, 7.0, 600)

target_species = ["Iris-versicolor", "Iris-virginica"]

fig, axes = plt.subplots(1, 2, figsize=(14, 4), sharey=True)

for ax, sp in zip(axes, target_species):
    # 1) Filter to species
    x_sp = data.loc[data["Class"] == sp, "Petal length"].dropna()

    # 2) Empirical params for this species (scalar!)
    m_sp = float(x_sp.mean())
    sigma_sp = float(x_sp.std(ddof=1))

    # 3) Conditional density model: Gaussian N(m_sp, sigma_sp^2)
    pdf_sp = norm.pdf(x_grid, loc=m_sp, scale=sigma_sp)

    # 4) Plot: normalized histogram + pdf
    ax.hist(x_sp, bins=bins, density=True, edgecolor="black", alpha=0.6, label="Normalized histogram")
    ax.plot(x_grid, pdf_sp, linewidth=2, label="Gaussian conditional pdf")
    ax.set_xlim(2, 7)
    ax.set_title(f"{sp}\n(m={m_sp:.3f}, σ={sigma_sp:.3f})")
    ax.set_xlabel("Petal length")
    ax.grid(alpha=0.3)
    ax.legend()

axes[0].set_ylabel("Probability density")
fig.suptitle("Conditional Distributions of Petal Length by Species", y=1.02)
plt.tight_layout()
plt.show()
```

### Why this code (code usage explanation)

* This is the standard “fit by moments” approach for a normal model.
* `float(...)` prevents the “sigma is a Series” bug.
* Histogram normalization ensures fair comparison to pdf.

### Part e Answer template

* EN: Within each species, the Gaussian fits better than the global single Gaussian (Part c), though tails may deviate.
* ZH: 分物种后的高斯通常更合理；整体混合时（Part c）更容易偏离高斯形状。

---

## Part f: Total probability mixture model (g_X(x)) ✅

### Total probability theorem (must be weighted sum!)

Correct formula:
$$
g_X(x)=P(V),f_{X|V}(x)+P(VC),f_{X|VC}(x)
$$
**Not** commas.

### Why mixture can outperform Part c

If the data is a mixture of subpopulations, the marginal distribution becomes a mixture (often multi-modal or skewed), which a single Gaussian cannot capture well.

### ✅ Code (paste into Part f code cell)

```python
from scipy.stats import norm

# ----- 0) Common settings (keep consistent with Part a/c/e) -----
bin_width = 0.1
bins = np.arange(2.0, 7.0 + bin_width + 1e-12, bin_width)
x_grid = np.linspace(2.0, 7.0, 600)

# ----- 1) Data used in Part a histogram over [2,7] -----
x_all = data["Petal length"].dropna()
x_all_in_range = x_all[(x_all >= 2.0) & (x_all <= 7.0)]

# ----- 2) Extract the two species samples (within [2,7]) -----
sp_v = "Iris-virginica"
sp_vs = "Iris-versicolor"

x_v = data.loc[data["Class"] == sp_v, "Petal length"].dropna()
x_vs = data.loc[data["Class"] == sp_vs, "Petal length"].dropna()

x_v = x_v[(x_v >= 2.0) & (x_v <= 7.0)]
x_vs = x_vs[(x_vs >= 2.0) & (x_vs <= 7.0)]

# ----- 3) Conditional densities: Gaussian per species -----
m_v, s_v = float(x_v.mean()), float(x_v.std(ddof=1))
m_vs, s_vs = float(x_vs.mean()), float(x_vs.std(ddof=1))

f_v = norm.pdf(x_grid, loc=m_v, scale=s_v)
f_vs = norm.pdf(x_grid, loc=m_vs, scale=s_vs)

# ----- 4) Weights: estimate P(V), P(VC) on the SAME domain [2,7] -----
# Rationale: Part a histogram only shows x in [2,7]. Using in-range weights keeps the target consistent.
count_v = len(x_v)
count_vs = len(x_vs)
w_v = count_v / (count_v + count_vs)
w_vs = count_vs / (count_v + count_vs)

# Mixture model g_X(x)
g = w_v * f_v + w_vs * f_vs

# ----- 5) Compare with Part c single Gaussian on the same target data -----
m_single = float(x_all_in_range.mean())
s_single = float(x_all_in_range.std(ddof=1))
single_pdf = norm.pdf(x_grid, loc=m_single, scale=s_single)

# ----- 6) Plot overlay -----
plt.figure(figsize=(7.5, 4.5))
plt.hist(x_all_in_range, bins=bins, density=True, edgecolor="black", alpha=0.55,
         label="Part a normalized histogram (x in [2,7])")
plt.plot(x_grid, g, linewidth=2.5,
         label=r"Mixture model $g_X(x)=P(V)f(x|V)+P(VC)f(x|VC)$")
plt.plot(x_grid, single_pdf, linewidth=2, linestyle="--", label="Single Gaussian (Part c)")

plt.xlim(2, 7)
plt.xlabel("Petal length")
plt.ylabel("Probability density")
plt.title("Part f: Mixture via Total Probability vs Histogram (and Part c Single Gaussian)")
plt.grid(alpha=0.3)
plt.legend()
plt.show()

print(f"Weights in [2,7]: P(virginica)={w_v:.3f}, P(versicolor)={w_vs:.3f}")
print(f"Virginica:    m={m_v:.4f}, σ={s_v:.4f}")
print(f"Versicolor:   m={m_vs:.4f}, σ={s_vs:.4f}")
print(f"Single Gauss: m={m_single:.4f}, σ={s_single:.4f}")
```

### Part f Answer template

* EN: The mixture model typically matches the histogram better than a single Gaussian because the overall distribution is the weighted combination of two class-conditional distributions.
* ZH: 混合模型通常比单高斯更贴合，因为总体分布本质上是两类条件分布的加权叠加。

---

# 3) Section 2: Two-dimensional joint modeling (before Section 3) 🧠

Section 2 introduces a **joint random vector**:
$$
(X,Y)=(L,W)=(\text{Petal length},\text{Petal width})
$$
and asks whether the joint distribution (or conditional joint distribution) can be modeled as multivariate normal.

---

## Example 1: Overall scatter plot (visualizing joint samples)

### Statistical meaning

Scatter plot visualizes the cloud of samples ($x_i,y_i$), which helps diagnose:

* single-cluster ellipse (suggesting Gaussian),
* multi-cluster mixture (suggesting mixtures or conditioning by class),
* linear trend (correlation).

### Reusable code snippet

```python
fig, ax = plt.subplots()
ax.scatter(data["Petal length"], data["Petal width"], alpha=0.3)
ax.axis("scaled")     # IMPORTANT: preserve geometry
ax.grid(True, alpha=0.3)
ax.set_xlabel("Petal length")
ax.set_ylabel("Petal width")
plt.show()
```

### Key code details

* `alpha`: reduces overplotting.
* `axis("scaled")`: otherwise ellipses get distorted and correlation visually misread.

---

## Example 2: Estimate mean vector, covariance, correlation (2D moments)

### Code pattern

```python
m = data[["Petal length", "Petal width"]].mean()
C = data[["Petal length", "Petal width"]].cov()
rho = data[["Petal length", "Petal width"]].corr(method="pearson")
```

### Math link

* Mean vector:
  $$
  \mu\approx \begin{bmatrix}\bar X\\bar Y\end{bmatrix}
  $$
* Covariance matrix:
  $$
  \Sigma \approx
  \begin{bmatrix}
  \mathrm{Var}(X) & \mathrm{Cov}(X,Y)\
  \mathrm{Cov}(X,Y) & \mathrm{Var}(Y)
  \end{bmatrix}
  $$
* Correlation:
  $$
  \rho_{XY}=\frac{\mathrm{Cov}(X,Y)}{\sigma_X\sigma_Y}
  $$

### Important usage detail

* `m` is a Series, `C` is a DataFrame.
  For SciPy, safest conversion:

```python
mu = m.to_numpy()
Sigma = C.to_numpy()
```

---

## Example 3: MVN contour overlay (fit joint Gaussian and draw level sets)

### What it does

1. Fit a multivariate normal ( \mathcal N(\mu,\Sigma) ) using sample mean/cov.
2. Evaluate the pdf on a grid.
3. Overlay **contour lines** on the scatter plot.

### Why contour lines are ellipses

For MVN, equal density sets satisfy:
$$
(\mathbf z-\mu)^T\Sigma^{-1}(\mathbf z-\mu)=\text{constant}
$$
which form ellipses (shape/rotation determined by (\Sigma)).

### Reusable code snippet (robust, recommended)

```python
from scipy.stats import multivariate_normal
import numpy as np
import matplotlib.pyplot as plt

axlim = [2, 7, 0, 3]  # example limits

X = data[["Petal length", "Petal width"]].dropna()
mu = X.mean().to_numpy()
Sigma = X.cov().to_numpy()

dist = multivariate_normal(mean=mu, cov=Sigma)

xs = np.linspace(axlim[0], axlim[1], 120)
ys = np.linspace(axlim[2], axlim[3], 120)
Xg, Yg = np.meshgrid(xs, ys)
pos = np.dstack((Xg, Yg))
Z = dist.pdf(pos)

fig, ax = plt.subplots()
ax.scatter(X["Petal length"], X["Petal width"], alpha=0.3)
ax.contour(Xg, Yg, Z, levels=8)
ax.axis("scaled")
ax.axis(axlim)
ax.grid(True, alpha=0.3)
ax.set_xlabel("Petal length")
ax.set_ylabel("Petal width")
ax.set_title("Scatter + fitted multivariate normal contours")
plt.show()
```

### Reading the plot (interpretation checklist)

* single elliptical cluster + contours match → MVN plausible
* two clusters → MVN poor globally; consider conditioning by class

---

## Example 4: Scatter by Class (conditional structure)

### Meaning

This visualizes conditional clouds:
$$
(X,Y)|C=c
$$
Often each class looks “more Gaussian” than the mixed dataset.

### Reusable code snippet

```python
fig, ax = plt.subplots()

for name, g in data.groupby("Class"):
    ax.scatter(g["Petal length"], g["Petal width"], alpha=0.3, label=name)

ax.axis("scaled")
ax.grid(True, alpha=0.3)
ax.set_xlabel("Petal length")
ax.set_ylabel("Petal width")
ax.legend(loc="upper left", bbox_to_anchor=(1,1))
ax.set_title("Scatter by species (Class)")
plt.show()
```

---

# 4) Part g–i: Species stats + classifier error curve ✅

---

## Part g: mean vectors, covariance matrices, correlation matrices by species

### ✅ Code

```python
cols = ["Petal length", "Petal width"]
groups = data.groupby("Class")

mean_by_species = {}
cov_by_species = {}
corr_by_species = {}

for sp, g in groups:
    X = g[cols].dropna()
    mean_by_species[sp] = X.mean()
    cov_by_species[sp] = X.cov()
    corr_by_species[sp] = X.corr(method="pearson")

for sp in mean_by_species:
    print("=" * 70)
    print(f"Species: {sp}")
    print("\nMean vector [E(L), E(W)]:")
    print(mean_by_species[sp])
    print("\nCovariance matrix:")
    print(cov_by_species[sp])
    print("\nCorrelation coefficient matrix:")
    print(corr_by_species[sp])

print("=" * 70)
print("Corr(Petal length, Petal width) by species:")
for sp in corr_by_species:
    r = corr_by_species[sp].loc["Petal length", "Petal width"]
    print(f"{sp}: {r:.6f}")
```

### Math link (what those outputs represent)

* $\mu_c$: mean vector for class (c)
* $\Sigma_c$: covariance matrix for class (c)
* $\rho_c$: correlation matrix for class (c)

### Key detail

Use the same column order consistently: `[Petal length, Petal width]` here.

---

## Part h: answer (based on Part g + scatter)

Paste this into the Part h markdown cell (then adjust if your computed numbers differ):

**(1) On average, which species has shorter petals?**

* EN: Versicolor has shorter petals on average (smaller mean petal length).
* ZH: Versicolor 平均花瓣更短（petal length 均值更小）。

**(2) Which species shows greater variation in petal length?**

* EN: Virginica shows greater variation in petal length (larger variance/standard deviation in the covariance matrix).
* ZH: Virginica 的 petal length 波动更大（方差/标准差更大）。

**(3) Which species has a larger correlation coefficient between petal length and width?**

* EN: Versicolor has a larger correlation coefficient between length and width (higher Pearson correlation).
* ZH: Versicolor 的 length–width 线性相关更强（Pearson 相关更大）。

---

## Part i: classifier error vs threshold T (D = 7W − L) ✅

### 4-line theory (why this works)

Assume for each class (c\in{VC,V}):
$$
(W,L)|c \sim \mathcal N(\mu_c,\Sigma_c)
$$
Define a linear score:
$$
D = 7W - L = a^T\begin{bmatrix}W \ L \end{bmatrix},\quad a=\begin{bmatrix}7\ -1\end{bmatrix}
$$
Then:
$$
D|c \sim \mathcal N(a^T\mu_c,\ a^T\Sigma_c a)
$$
Classifier: predict virginica if $D>T$. Error probability:
$$
P_e(T)=\pi_{VC},P(D>T|VC) + \pi_V,P(D\le T|V)
$$
where $\pi_c=P(C=c)$ (empirical priors from counts).

### ✅ Code (robust, with one-feature baseline comparison)

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import norm

W_col = "Petal width"
L_col = "Petal length"
C_col = "Class"

sp_vs = "Iris-versicolor"
sp_v  = "Iris-virginica"

df = data.loc[data[C_col].isin([sp_vs, sp_v]), [C_col, W_col, L_col]].dropna()

# Priors
n_vs = (df[C_col] == sp_vs).sum()
n_v  = (df[C_col] == sp_v).sum()
pi_vs = n_vs / (n_vs + n_v)
pi_v  = n_v  / (n_vs + n_v)

# Estimate mu, Sigma for (W, L) in this order
def mean_cov_2d(subdf):
    X = subdf[[W_col, L_col]].to_numpy()
    mu = X.mean(axis=0)
    Sigma = np.cov(X.T, ddof=1)
    return mu, Sigma

mu_vs, Sigma_vs = mean_cov_2d(df[df[C_col] == sp_vs])
mu_v,  Sigma_v  = mean_cov_2d(df[df[C_col] == sp_v])

# Linear transform D = 7W - L
a = np.array([7.0, -1.0])

mD_vs = float(a @ mu_vs)
sD_vs = float(np.sqrt(a @ Sigma_vs @ a))

mD_v  = float(a @ mu_v)
sD_v  = float(np.sqrt(a @ Sigma_v @ a))

# Error curve
T_vals = np.linspace(0.0, 15.0, 1501)  # step=0.01
P_D_gt_T_vs = norm.sf(T_vals, loc=mD_vs, scale=sD_vs)  # P(D>T | versicolor)
P_D_le_T_v  = norm.cdf(T_vals, loc=mD_v,  scale=sD_v)  # P(D<=T | virginica)

P_err = pi_vs * P_D_gt_T_vs + pi_v * P_D_le_T_v

idx = int(np.argmin(P_err))
T_star = float(T_vals[idx])
P_star = float(P_err[idx])

plt.figure(figsize=(7.5, 4.5))
plt.plot(T_vals, P_err, linewidth=2)
plt.axvline(T_star, linestyle="--")
plt.xlabel("Threshold T")
plt.ylabel("Probability of error")
plt.title("Classifier error vs threshold (Gaussian class-conditionals)")
plt.grid(alpha=0.3)
plt.show()

print(f"Priors: P(versicolor)={pi_vs:.4f}, P(virginica)={pi_v:.4f}")
print("D|class parameters:")
print(f"  versicolor: mean={mD_vs:.4f}, std={sD_vs:.4f}")
print(f"  virginica:  mean={mD_v:.4f},  std={sD_v:.4f}")
print(f"Optimal threshold (grid): T*={T_star:.4f}")
print(f"Minimum error probability: P_err(T*)={P_star:.6f}")

# ----- baseline: one feature alone (L only) -----
L_vs = df.loc[df[C_col] == sp_vs, L_col].to_numpy()
L_v  = df.loc[df[C_col] == sp_v,  L_col].to_numpy()

mL_vs, sL_vs = float(L_vs.mean()), float(L_vs.std(ddof=1))
mL_v,  sL_v  = float(L_v.mean()),  float(L_v.std(ddof=1))

t_vals = np.linspace(2.0, 7.0, 2001)
P_L_gt_t_vs = norm.sf(t_vals, loc=mL_vs, scale=sL_vs)
P_L_le_t_v  = norm.cdf(t_vals, loc=mL_v,  scale=sL_v)

P_err_1d = pi_vs * P_L_gt_t_vs + pi_v * P_L_le_t_v

j = int(np.argmin(P_err_1d))
t_star = float(t_vals[j])
P_star_1d = float(P_err_1d[j])

print("\n[1D baseline using L only]")
print(f"Optimal length threshold t*={t_star:.4f}")
print(f"Minimum error (L only) = {P_star_1d:.6f}")
print(f"Improvement (1D - 2D)  = {P_star_1d - P_star:.6f}")
```

### Part i Answer template (fill with your printed values)

* EN:

  * The optimal threshold is $T^*=$ **(printed value)** with minimum error $P_e(T^*)=$ **(printed value)**.
  * The error is typically lower than using one feature alone (compare with `Minimum error (L only)`).
* ZH:

  * 最优阈值 $T^*=$ **(你的输出)**，最小错误率 $P_e(T^*)=$ **(你的输出)**。
  * 一般会比只用 (L) 的单特征分类器更低（比较两者最小错误率）。

---

# 5) Bugbook (Most common mistakes) 🧯

## Bug 1: `norm.pdf` broadcasting error (shapes (600,) vs (2,))

**Symptom:** ValueError about broadcasting.
**Cause:** `m` or `sigma` is a vector/Series (e.g., computed from a DataFrame with multiple columns).
**Fix:** force scalars:

```python
m = float(x.mean())
sigma = float(x.std(ddof=1))
```

## Bug 2: Comparing histogram to pdf without `density=True`

**Symptom:** pdf curve seems tiny or mismatch scale.
**Cause:** histogram is counts; pdf is density.
**Fix:** always `density=True`.

## Bug 3: Mixed bin settings between plots

**Symptom:** histograms look “different” but due to different bin widths.
**Fix:** reuse the same `bins` across comparisons.

## Bug 4: `axis('scaled')` missing for scatter/contour

**Symptom:** ellipse looks stretched; correlation visual is misleading.
**Fix:** always `ax.axis('scaled')`.

---

# Appendix A: Reusable Code Snippets (Toolbox) ♻️

## A.1 Global plotting constants

```python
BIN_WIDTH = 0.1
BINS = np.arange(2.0, 7.0 + BIN_WIDTH + 1e-12, BIN_WIDTH)
XGRID = np.linspace(2.0, 7.0, 600)
```

## A.2 Overlay histogram and an arbitrary pdf curve

```python
def overlay_hist_and_pdf(ax, x, bins, x_grid, pdf_vals, title=""):
    x = pd.Series(x).dropna()
    ax.hist(x, bins=bins, density=True, alpha=0.6, edgecolor="black", label="Normalized histogram")
    ax.plot(x_grid, pdf_vals, linewidth=2, label="pdf")
    ax.set_xlim(x_grid.min(), x_grid.max())
    ax.set_title(title)
    ax.grid(alpha=0.3)
    ax.legend()
```

## A.3 Fit Gaussian by moments and return pdf on grid

```python
from scipy.stats import norm

def gaussian_pdf_from_samples(x, x_grid, ddof=1):
    x = pd.Series(x).dropna()
    m = float(x.mean())
    s = float(x.std(ddof=ddof))
    return m, s, norm.pdf(x_grid, loc=m, scale=s)
```

## A.4 Scatter + MVN contour (joint Gaussian)

```python
from scipy.stats import multivariate_normal

def scatter_with_mvn_contour(df, xcol, ycol, axlim, levels=8, alpha=0.3, grid_n=120):
    X = df[[xcol, ycol]].dropna()
    mu = X.mean().to_numpy()
    Sigma = X.cov().to_numpy()
    dist = multivariate_normal(mean=mu, cov=Sigma)

    xs = np.linspace(axlim[0], axlim[1], grid_n)
    ys = np.linspace(axlim[2], axlim[3], grid_n)
    Xg, Yg = np.meshgrid(xs, ys)
    pos = np.dstack((Xg, Yg))
    Z = dist.pdf(pos)

    fig, ax = plt.subplots()
    ax.scatter(X[xcol], X[ycol], alpha=alpha)
    ax.contour(Xg, Yg, Z, levels=levels)
    ax.axis("scaled")
    ax.axis(axlim)
    ax.grid(True, alpha=0.3)
    ax.set_xlabel(xcol); ax.set_ylabel(ycol)
    ax.set_title("Scatter + fitted MVN contour")
    plt.show()

    return mu, Sigma
```

## A.5 Generic group mean/cov/corr utility

```python
def group_mean_cov_corr(df, group_col, cols):
    out = {}
    for name, g in df.groupby(group_col):
        X = g[cols].dropna()
        out[name] = {
            "mean": X.mean(),
            "cov": X.cov(),
            "corr": X.corr(method="pearson")
        }
    return out
```

---

End of notes.