## ✅ Part a 代码（Normalized Histogram, bin=0.1, x∈[2,7]）

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

---

## 为什么要这么写（你写的每一行在干嘛）🧠

### EN（point-by-point）

* `x = data["Petal length"].dropna()`
  把目标变量取出来，并移除缺失值，避免 `hist` 因 NaN 报错/出怪图。
* `bins = np.arange(2.0, 7.0 + bin_width + 1e-12, bin_width)`
  **关键**：bin size=0.1 必须“硬控”。

  * `np.arange(2, 7, 0.1)` 有时会因为浮点误差导致最后一个边界不是精确的 7.0
  * 所以我加了 `+ bin_width + 1e-12`，确保 **7.0 一定在 bin 边界里**。
* `plt.hist(..., density=True)`
  **Normalized histogram 的核心**。
  `density=True` 让柱子的高度变成“概率密度”，满足：
  [
  \text{height}_k = \frac{\text{count}_k}{N \cdot \Delta}
  ]
  其中 (N) 是样本数，(\Delta) 是 bin width。这样整个柱状图的 **面积积分=1**。
* `plt.xlim(2,7)`
  强制显示题目要求的区间，避免数据在区间外把视觉尺度拉歪。
* `edgecolor/grid/labels/title`
  让图符合作业可读性标准（不影响数学定义，但影响评分观感 😐）。

### 中文（逐点对齐）

* `dropna()`：清掉缺失值，防止直方图异常。
* `np.arange(...)`：严格指定 **bin 宽 0.1**，并用 `+1e-12` 处理浮点误差，保证 **右端点 7 能包含进 bins**。
* `density=True`：把“计数直方图”变成“概率密度直方图”，柱高就是
  (\text{count}/(N\cdot \Delta))，整体面积为 1 —— 这就是 normalized histogram 的定义。
* `xlim(2,7)`：保证只画题目要求区间。
* 其余：提高可读性。

---

## 以后画 normalized histogram 要记住的坑（Checklist）✅🧷

### 1) **density=True 的含义不是“每根柱子和为 1”**

* 它保证的是：**所有柱子的面积之和 = 1**
* 如果你改了 bin width（比如从 0.1 改成 0.2），柱子的高度会跟着变（因为是密度）。

### 2) **bin edges 最好自己生成，不要只给 bins=50 这种**

* 作业/论文里通常要求固定 bin width（0.1 这种），那就用 `np.arange` 明确指定。

### 3) **浮点误差会让边界出 bug**

* 常见现象：你以为到 7.0 了，其实最后一个边界是 6.9999999997
* 处理方式：像我这样加一个极小量 `+1e-12`，或者用 `np.linspace`（但 linspace 更适合固定 bin 数）。

### 4) **比较直方图 vs PDF 曲线时：必须是 density histogram**

* 你 Part c / e / f 要叠加 `pdf` 曲线时，直方图必须是 density=True
  否则量纲不一致：一个是“计数”，一个是“密度”，叠一起毫无意义。

### 5) **样本数不同的组（分 species）更要用 density**

* 不同类样本量可能不同，用 density 才能公平比较“形状”。


下面是 **Part b** 该写的内容（直接放到 Part b 对应的 code cell 里即可），以及**为什么这么写** & **需要记住的点** ✅

---

## ✅ Part b 代码：经验均值 (m) 与经验标准差 (\sigma)

```python
# Part b: empirical mean m and empirical standard deviation sigma of petal length (all species)

x = data["Petal length"].dropna()   # remove NaN if any

m = x.mean()
sigma = x.std(ddof=1)              # ddof=1 -> denominator (n-1), matches the formula in the prompt

print(f"Empirical mean m = {m:.6f}")
print(f"Empirical std  σ = {sigma:.6f}")
```

> 你也可以用 `data[["Petal length"]].mean()` / `.std()`（DataFrame 版本），但对单列来说 `Series` 写法更直接。

---

## 为什么要这样写 🧠

### EN (aligned)

* `dropna()`
  Avoids NaNs contaminating the mean/std calculation.
* `m = x.mean()`
  Pandas computes (\frac{1}{n}\sum r_i) by default for mean.
* `sigma = x.std(ddof=1)`
  The assignment’s formula uses (\frac{1}{n-1}\sum (r_i-m)^2).
  In pandas, `std()` uses **sample standard deviation** by default (i.e., `ddof=1`), but I set it explicitly to match the prompt and avoid ambiguity.

### 中文（逐点对齐）

* `dropna()`：避免缺失值 NaN 影响统计量（会导致结果变成 NaN 或被忽略得不明确）。
* `mean()`：就是题面给的经验均值公式。
* `std(ddof=1)`：题面明确写的是 **(n-1)**（样本标准差），所以要用 `ddof=1`。
  Pandas 默认也通常是 `ddof=1`，但**显式写出来更稳**，防止你以后换库/换同学代码口径不一致。

---

## 以后做均值/标准差要记住的坑（Checklist）✅

1. **标准差分母是 (n) 还是 (n-1)**

* 题目给的是：(\sigma=\sqrt{\frac{1}{n-1}\sum (r_i-m)^2})
* 对应 pandas：`std(ddof=1)`
* 如果你写 `ddof=0`，那就是“总体标准差”（分母 (n)），会和作业要求不一致。

2. **Series vs DataFrame**

* `data["Petal length"]` 是 **Series**（最常用）
* `data[["Petal length"]]` 是 **DataFrame**（返回表格；mean/std 返回也是表格）

3. **缺失值处理要统一**

* pandas 的 `mean/std` 会自动跳过 NaN（skipna=True），但你显式 `dropna()` 能让流程更透明，避免后面你用 numpy 时口径不同。

# Section 2
下面我按你们 `Programming Assignment 1.ipynb` 的 **Section 2**（从 “Section 2: Working with two joint random variables …” 一直到 **Part g** 前）逐段解释：**这些例子在做什么、对应哪些统计学概念、代码怎么用、以及可复用代码模板/经验**。重点放在代码使用与数学意义的对应关系 ✅📌

---

## 0) Section 2 总目标是什么？

你们在 Section 1 里研究的是 **一维随机变量** (X=) petal length。
Section 2 扩展到 **二维联合随机变量**：

[
(X,Y) = (\text{Petal length},\ \text{Petal width})
]

要解决的核心问题是：

* 数据在二维平面里是什么形状？（散点图）
* (X,Y) 的均值向量、协方差矩阵、相关系数矩阵是什么？（用 pandas 估计）
* 用一个 **二维高斯（多元正态）** ( \mathcal N(\mu,\Sigma) ) 拟合联合分布，并把其 **等密度线**（contour）叠加到散点图上，直观看拟合好不好。
* 再按 species 分组看：不同物种的分布结构是否不同（groupby + scatter）。

---

## 1) 例子 1：整体散点图（cell 26）

你们的代码（概念上）：

```python
fig, ax = plt.subplots()
ax.scatter(data['Petal length'], data['Petal width'], alpha=0.3)
...
```

### 这在统计上是什么？

* 散点图是在观察 **联合样本 ((x_i,y_i))** 的几何结构：

  * 是否近似椭圆形（暗示“可能接近高斯”）
  * 是否明显多簇（暗示“混合分布”）
  * 是否存在线性趋势（暗示相关性）

### 代码使用重点 ✅

* `alpha=0.3`：点多时避免遮挡（overplotting）
* `axlim = [2,7,0,3]`：固定坐标范围，让不同图可比（很重要）
* `ax.axis('scaled')`（你们在后面按组图里用到了）：让 x/y 轴单位长度一致，不然椭圆会被拉伸，看起来“相关性”被误判。

#### 可复用模板：二维散点图（全体/任意两列）

```python
def scatter_xy(df, xcol, ycol, axlim=None, title=None, alpha=0.3):
    import matplotlib.pyplot as plt
    fig, ax = plt.subplots()
    ax.scatter(df[xcol], df[ycol], alpha=alpha)
    if axlim is not None:
        ax.axis(axlim)
    ax.axis('scaled')
    ax.set_xlabel(xcol); ax.set_ylabel(ycol)
    ax.grid(True, alpha=0.3)
    if title: ax.set_title(title)
    plt.show()
```

---

## 2) 例子 2：估计均值向量、协方差矩阵、相关系数矩阵（cell 28）

你们代码核心：

```python
m = data[['Petal length','Petal width']].mean()
C = data[['Petal length','Petal width']].cov()
rho = data[['Petal length','Petal width']].corr(method='pearson')
```

### 数学对应关系（统计学知识点）🧠

#### (1) 均值向量（mean vector）

$$
\mu =
\begin{bmatrix}
\mu_X\
\mu_Y
\end{bmatrix}
\approx
\begin{bmatrix}
\bar X\
\bar Y
\end{bmatrix}
$$

`pandas.mean()` 对每列算样本均值（分母 (n)）。

#### (2) 协方差矩阵（covariance matrix）

$$
\Sigma =
\begin{bmatrix}
\mathrm{Var}(X) & \mathrm{Cov}(X,Y)\
\mathrm{Cov}(X,Y) & \mathrm{Var}(Y)
\end{bmatrix}
$$

`pandas.cov()` 默认给的是 **样本协方差**（分母 (n-1)），对应你们前面 std 的口径（`ddof=1`）一致 ✅

协方差解释：

* (\mathrm{Cov}(X,Y)>0)：大体同涨同跌（正线性关系）
* (\mathrm{Cov}(X,Y)<0)：此涨彼跌（负线性关系）
* (\mathrm{Cov}(X,Y)\approx 0)：线性关系弱（但不代表独立）

#### (3) 相关系数矩阵（correlation matrix）

[
\rho_{XY} = \frac{\mathrm{Cov}(X,Y)}{\sigma_X \sigma_Y}
]

`pandas.corr(method="pearson")` 给 Pearson 相关（线性相关），范围 ([-1,1])，方便比较“强弱”，不受量纲影响。

### 代码使用细节 ✅

* `m` 是 **Series（长度2）**，`C` 和 `rho` 是 2×2 DataFrame。
* 后续喂给 `multivariate_normal(m, C)` 时一般能自动转换，但更稳的写法是：

  * `mu = m.values` 或 `mu = m.to_numpy()`
  * `Sigma = C.values`

#### 可复用模板：一次性输出均值/协方差/相关矩阵

```python
def summarize_joint(df, cols):
    X = df[cols].dropna()
    mu = X.mean()
    Sigma = X.cov()          # sample covariance (n-1)
    Corr = X.corr('pearson')
    return mu, Sigma, Corr
```

---

## 3) 例子 3：叠加二维高斯拟合的等密度线 contour（cell 30）

你们代码做了这件事：

1. 用估计得到的 (\mu=m,\ \Sigma=C) 构造二维正态分布
2. 在网格点上计算 pdf
3. 用 `contour` 画等高线叠加到 scatter 上

### 统计学含义 🧠

你们是在做一个经典近似建模：

$$
(X,Y) \approx \mathcal N(\mu,\Sigma)
$$

* 二维高斯的等密度线是**椭圆**（由 (\Sigma) 决定椭圆的旋转方向与“扁/圆”程度）
* 如果散点云大体呈椭圆形、单簇、对称，则这个模型通常还行
* 若散点明显多簇（比如混合了多个 species），单个二维高斯往往不够好 → 这为后面按 species 建模做铺垫

### 代码使用重点（你们这段最“工程化”的部分）✅

#### (1) `multivariate_normal(m, C)`

* `multivariate_normal` 是多元正态对象（SciPy）
* `.pdf(pos)` 会输出每个网格点上的密度值

#### (2) `meshgrid` & `dstack`

* `np.meshgrid` 把 x、y 轴采样点扩展成网格
* `pos = np.dstack((x, y))` 把每个网格点变成形如 `(x,y)` 的向量，给 pdf 批量计算

#### (3) 网格分辨率是关键 trade-off

* `np.linspace(axlim[0], axlim[1])` 默认 50 个点左右（SciPy/NumPy默认），通常够用
* 网格越密：曲线更平滑，但计算更慢（尤其三维会很慢）

#### 可复用模板：scatter + MVN contour 一键出图

```python
from scipy.stats import multivariate_normal
import numpy as np
import matplotlib.pyplot as plt

def scatter_with_mvn_contour(df, xcol, ycol, axlim, levels=8, alpha=0.3, grid_n=120):
    X = df[[xcol, ycol]].dropna()
    mu = X.mean().to_numpy()
    Sigma = X.cov().to_numpy()

    dist = multivariate_normal(mean=mu, cov=Sigma)

    x = np.linspace(axlim[0], axlim[1], grid_n)
    y = np.linspace(axlim[2], axlim[3], grid_n)
    Xg, Yg = np.meshgrid(x, y)
    pos = np.dstack((Xg, Yg))
    Z = dist.pdf(pos)

    fig, ax = plt.subplots()
    ax.scatter(X[xcol], X[ycol], alpha=alpha)
    ax.contour(Xg, Yg, Z, levels=levels)
    ax.axis('scaled')
    ax.axis(axlim)
    ax.set_xlabel(xcol); ax.set_ylabel(ycol)
    ax.grid(True, alpha=0.3)
    ax.set_title("Scatter + fitted MVN contour")
    plt.show()

    return mu, Sigma
```

---

## 4) 例子 4：按 species 分组画散点图（cell 32）

你们代码：

```python
for name, groups in data.groupby('Class'):
    ax.scatter(x='Petal length', y='Petal width', data=groups, label=name, alpha=0.3)
```

### 统计学意义 🧠

这是在观察 **条件分布** 的样本结构：

$$
(X,Y)\mid C=c
$$

你会看到不同 species 的均值位置不同、扩散不同、相关性不同。
这就是你们后面 Part g/h/i 要做的逻辑基础：**同一总体数据混在一起会变复杂，但分组后可能更接近高斯**（每类单峰椭圆）。

### 代码使用重点 ✅

* `groupby('Class')`：按类别切分数据
* `label=name + legend`：用于识别不同簇
* `bbox_to_anchor=(1,1)`：把 legend 挪到图外，避免遮挡数据点（好习惯）

#### 可复用模板：按组 scatter（legend 外置）

```python
def scatter_by_group(df, xcol, ycol, group_col, axlim=None, alpha=0.3):
    import matplotlib.pyplot as plt
    fig, ax = plt.subplots()
    for gname, gdf in df.groupby(group_col):
        ax.scatter(gdf[xcol], gdf[ycol], alpha=alpha, label=gname)
    if axlim is not None:
        ax.axis(axlim)
    ax.axis('scaled')
    ax.set_xlabel(xcol); ax.set_ylabel(ycol)
    ax.grid(True, alpha=0.3)
    ax.legend(loc='upper left', bbox_to_anchor=(1,1))
    plt.show()
```

---

## 5) 这些图“怎么用”？它们在为 Part g/h/i 铺路

Section 2 的例子不是随便画图，它们在建立一个建模链条：

1. **整体 scatter**：发现混合后可能多簇 → 单一模型可能不好
2. **整体 mean/cov/corr**：把二维数据用二阶矩摘要（(\mu,\Sigma,\rho)）
3. **整体 MVN contour**：检验“整体用一个二维高斯”是否合理（通常对混合类不理想）
4. **分组 scatter**：发现每组更像椭圆单簇 → “每组一个二维高斯”更合理
5. 于是 Part g 要你算：**每个 species 的 (\mu_c,\Sigma_c,\rho_c)**
6. Part i 才能基于：((X,Y)|C=c \sim \mathcal N(\mu_c,\Sigma_c)) 去算分类器误差概率

---

## 6) 实战级坑点清单（你们后面最可能踩的）⚠️✅

1. **pandas 的 mean/cov/corr 是按列算的**

   * 传给 SciPy 时要保证维度是 (2,) 和 (2,2)，必要时 `.to_numpy()`
2. **样本 vs 总体口径**

   * `cov()` 默认 (n-1)，与 `std(ddof=1)` 一致
3. **axis scaling**

   * 不 `axis('scaled')` 你会“看错相关性强弱”
4. **多簇 ≠ 单高斯**

   * 混合类别整体分布常常不是椭圆单峰
5. **网格密度**

   * `grid_n` 太大就慢；太小 contour 变粗糙