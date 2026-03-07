---
title: 数据科学：Scipy、Scikit-Learn笔记
date: 2026-01-04 08:12:00
tags:
  - "数据科学"
  - "Scikit-Learn"
  - "Scipy"
  - "笔记"
categories:
  - "数据科学"
---

﻿[**数据科学：Numpy、Pandas笔记**](https://blog.csdn.net/qq_45832050/article/details/127466841)

[**数据科学：Matplotlib、Seaborn笔记**](https://blog.csdn.net/qq_45832050/article/details/134764886)
## [超参数调优：网格搜索，贝叶斯优化（optuna）详解](https://blog.csdn.net/qq_45832050/article/details/138012101)
## [XGBoost原生接口和Sklearn接口参数详解](https://blog.csdn.net/qq_45832050/article/details/138009902)
## [LightGBM原生接口和Sklearn接口参数详解](https://blog.csdn.net/qq_45832050/article/details/137434400)


@[TOC](数据科学：Numpy、Pandas、Matplotlib、Seaborn、Scipy、Scikit-Learn)

## cvxpy

```python
import cvxpy as cp
import numpy as np
from sklearn.datasets import load_boston
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# 加载数据
boston = load_boston()
X = boston.data
y = boston.target

# 数据标准化
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2, random_state=42)

# 定义优化变量（线性回归的系数）
beta = cp.Variable(X_train.shape[1])

# 定义目标函数（最小化平方误差和）
objective = cp.Minimize(cp.sum_squares(y_train - X_train @ beta))

# 定义系数约束（例如，beta[0] >= 0, beta[1] <= 5）
constraints = [beta[0] >= 0, beta[1] <= 5, beta[2] >= 0, beta[3] <= 5, beta[4] >= 0, beta[5] <= 5,
               beta[6] >= 0, beta[7] <= 5, beta[8] >= 0, beta[9] <= 5, beta[10] >= 0, beta[11] <= 5, beta[12] <= 5]

# 定义问题
# prob = cp.Problem(objective, constraints)
prob = cp.Problem(objective)

# 求解问题
prob.solve()

# 获取最优系数
optimal_beta = beta.value
# 输出结果
print("Optimal coefficients:", optimal_beta)
# 使用最优系数进行预测
y_pred = X_test @ optimal_beta

# 评估模型性能（例如，计算均方误差）
mse = np.mean((y_test - y_pred) ** 2)
print("Mean Squared Error:", mse)
```

## Scipy
模块|作用
|--|--|
scipy.cluster	|矢量量化/Kmeans
scipy.constants|	物理和数学常数
scipy.fftpack	|傅里叶变换
scipy.integrate|	积分
scipy.interpolate|	插值
http://scipy.io	|数据输入输出
scipy.linalg	|线性代数
scipy.ndimage|	n维图像包
scipy.odr	|Orthogonal distance regression
scipy.optimize	|优化
scipy.signal	|信号处理
scipy.sparse|	稀疏矩阵
scipy.spatial|	空间数据结构和算法
scipy.special	|任何特殊的数学函数
scipy.stats	|统计数据

### stats
```python
from scipy.stats import norm
```
**常用函数**

 - norm.cdf 返回对应的累计分布函数值
 - norm.pdf 返回对应的概率密度函数值
 - norm.rvs 产生指定参数的随机变量
 - norm.fit 返回给定数据下，各参数的最大似然估计（MLE）值

### savgol_filter
实现了Savitzky-Golay滤波器算法，这是一种用于平滑数据序列的方法，特别适用于去除噪声同时保留原始信号的关键特征如峰值、谷值等。
**参数详解**

 - x: 输入数据数组，可以是一维或二维的。如果输入是二维的，那么每一行都将被视为一个独立的数据序列。
 -  window_length: 窗口的长度，必须是一个正奇数。这决定了滤波器的窗口大小，也就是进行多项式拟合时所用的数据点的数量。 
 - polyorder: 多项式的阶数，用于拟合窗口内的数据点。多项式的阶数必须小于窗口长度。
 -  deriv (可选): 表示想要计算的导数阶数，默认为0，表示不求导数，输出就是平滑后的数据。 
 - delta (可选): 采样间距，默认为1.0。如果deriv非零，这个值用来缩放导数。
 -  axis (可选):  在多维数组中指定沿哪个轴进行滤波，默认为-1（最后一个轴）。
 -  mode (可选):  端点模式，定义了边界之外的数据处理方式。可选项包括'interp', 'nearest', 'mirror', 或'reflect'等。
 -  cval (可选): 当mode是'constant'时，指定边界外的值。

**工作原理**
Savitzky-Golay滤波器的基本思想是在每个数据点周围取一个窗口，然后在这个窗口内使用一个较低阶次的多项式对数据进行拟合。拟合过程中使用最小二乘法来找到最佳多项式。这个多项式可以用来估计窗口中心点处的值，这个估计值就成为平滑后的数据点。
相比于简单的移动平均，Savitzky-Golay滤波器能够更好地保留信号的特征，比如尖峰和拐点，因为它不仅仅是一个简单的平均过程，而是一个基于多项式拟合的过程。

## Scikit-Learn
### 聚类
#### K-Means (KMeans)
**优点**:

 - 效率高：算法简单，时间复杂度较低（O(n * K * I *d)，其中n是样本数，K是簇数，I是迭代次数，d是特征维度），尤其适用于大规模数据集。
 - 结果可解释：每个样本被明确分配到一个簇，簇中心具有实际意义（各簇的均值向量）。
 - 并行化友好：更新簇中心的过程可以并行计算，进一步提升处理速度。

**缺点**:

 - 需要预先指定簇数K：实际应用中可能难以确定最优的K值，通常需要结合领域知识、试错法（如肘部法则、轮廓系数）或模型选择方法来确定。
 - 对初始簇中心敏感：不同的初始化可能导致不同的聚类结果，通常通过多次运行（设置n_init参数）并取最优解来缓解这个问题。
 - 假设簇为凸形且大小相近：对于非球形、大小差异大或分布不均匀的簇，K-Means可能无法准确划分。
 - 对异常值敏感：异常值可能显著影响簇中心位置，导致聚类质量下降。

K-means聚类算法通常受益于数据预处理中的归一化步骤，特别是当数据特征具有显著不同的尺度或者单位时。

在K-means聚类中，轮廓系数可以帮助我们选择合适的簇数（K值）

```python
import numpy as np
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

# 假设我们有一个名为data的数据集，它是一个二维numpy数组，包含了需要聚类的样本
data = ...

# 要尝试的K值范围
k_values = range(2, 11)

# 存储轮廓系数的列表
silhouette_scores = []

# 遍历K值范围，计算每个K值对应的轮廓系数
for k in k_values:
    # 创建KMeans模型并拟合数据
    kmeans = KMeans(n_clusters=k, random_state=42).fit(data)

    # 计算轮廓系数
    score = silhouette_score(data, kmeans.labels_)
    silhouette_scores.append(score)

# 绘制轮廓系数随K值变化的曲线
plt.plot(k_values, silhouette_scores, marker='o')
plt.xlabel('Number of clusters (K)')
plt.ylabel('Silhouette Score')
plt.title('Silhouette Coefficient for Different K Values')
plt.grid(True)
plt.xticks(k_values)
plt.show()

# 找出最大轮廓系数对应的K值
best_k = k_values[np.argmax(silhouette_scores)]
print(f"The optimal number of clusters is {best_k} based on the maximum silhouette score.")
```
#### DBSCAN
**优点：**

 - 自动识别簇数：DBSCAN不需要用户预先指定簇的数量，而是根据数据的内在密度分布自动发现簇，特别适用于不知道数据中究竟有多少个簇的情况。
 - 适应任意形状的簇：DBSCAN能够有效地处理任意形状（包括非凸形、不规则形）的簇，对簇的大小、密度和分布形态没有严格限制，尤其适用于对复杂、不规则数据集的聚类。
 - 对噪声和离群点的鲁棒性：DBSCAN能够识别并标记低密度区域的点为噪声点，不会将它们强行纳入任何簇中。这意味着离群点和噪声不会影响簇的边界和形状，提高了聚类结果的稳健性。
 - 基于密度的聚类：DBSCAN基于数据点之间的邻近性和密度进行聚类，能够捕捉到数据的内在结构和模式，对于密度差异明显的数据集有较好的聚类效果。

**缺点：**

 - 对参数敏感：DBSCAN有两个关键参数：eps（邻域半径）和min_samples（形成核心点所需的邻居数）。这两个参数的选择对聚类结果有重大影响，选择不当可能导致过分割（将一个簇划分为多个小簇）或欠分割（多个簇被合并为一个簇）。选择合适参数可能需要领域知识、试错或可视化辅助。
 - 对高维数据和大规模数据的处理能力：随着数据维度的增加，定义合适的邻域半径（eps）变得困难，因为“距离”的概念在高维空间中往往失去意义（“维度灾难”）。此外，尽管DBSCAN理论上的时间复杂度可以接受，但在实践中，对于大规模数据集，其邻域搜索过程可能变得相当耗时。
 - 对数据分布的均匀性要求：DBSCAN假设数据分布有一定的均匀性，即簇内密度高于簇间密度。若数据中存在密度分布不均匀、簇间密度差距较小的情况，DBSCAN可能无法准确区分不同簇。
 - 对异常值的处理：虽然DBSCAN能够识别并标记噪声点，但如果异常值非常密集或异常值区域与正常数据区域的密度差异不大，DBSCAN可能无法有效区分噪声与正常数据，导致聚类结果受到影响。
 - 仅适用于数值型数据：与K-means类似，DBSCAN基于欧氏距离计算，只能应用于数值型特征。对于非数值型数据（如类别、文本、时间序列等），需要进行适当的预处理（如编码、转换）才能应用DBSCAN。

综上所述，sklearn中的DBSCAN是一个强大的密度-based聚类工具，特别适合于处理簇数量未知、簇形状复杂且对噪声和离群点有较强鲁棒性需求的数值型数据集。然而，对于高维、大规模数据、密度分布不均匀或参数选择困难的场景，可能需要结合其他预处理技术、参数调优策略或考虑使用其他聚类算法。

**参数**

DBSCAN有两个关键参数，它们决定了簇的识别方式和结果：

 - eps（邻域半径）

定义：eps是一个距离阈值，它决定了一个点的邻域范围。对于给定的数据点，其邻域是指所有与该点距离小于或等于eps的点的集合。
作用：eps直接影响到“邻近”或“相邻”的概念，即在eps范围内被认为是“邻近”的点。簇是通过连接足够数量的邻近点形成的。
选择：eps的选择应当依据数据的实际分布和密度。过小的eps可能导致过分割（将一个簇划分为多个小簇），而过大的eps可能导致欠分割（多个簇被合并为一个簇）。通常需要通过探索性数据分析（如绘制距离分布图、观察数据点在二维或三维空间的分布）或试错法来确定合适的eps值。

 - min_samples（形成核心点所需的邻居数）

定义：min_samples是一个整数，表示构成一个“核心点”所需的邻域内点的最小数量。如果一个点的邻域内至少有min_samples个点（包括自身），则称这个点为核心点。
作用：min_samples决定了簇的“密度”要求。只有当一个点的邻域内点数达到或超过min_samples时，该点才会被认为是簇的一部分。簇是由核心点及其直接或间接邻域内的点组成。
选择：min_samples的选择反映了对簇内密度的主观判断。较小的值可能导致簇边界模糊，较大的值可能导致簇过于稀疏。通常需要根据数据分布特点和对簇内紧密度的要求来设定。选择时可以考虑与eps一起调整，以达到最佳聚类效果。

除了这两个主要参数外，sklearn库实现的DBSCAN还可能包含以下参数：
metric：定义用于计算两点间距离的度量方法，默认为欧氏距离。可以根据数据特性选择其他距离度量，如曼哈顿距离、余弦相似度等。
metric_params：当使用自定义距离度量时，可以通过此参数传递额外的度量参数。
algorithm：选择用于构建邻域的算法。常见的选项有'auto'（自动选择）、'ball_tree'、'kd_tree'或'brute'（暴力搜索）。对于大规模数据，高效的索引结构（如ball_tree或kd_tree）可以显著提高性能。
leaf_size：当使用树型结构（如ball_tree或kd_tree）时，指定叶子节点包含的最大样本数。该值影响索引的构建时间和空间复杂度，以及查询效率。
p：仅当使用minkowski距离作为度量时有效，表示距离度量的幂次（如p=2对应欧氏距离，p=1对应曼哈顿距离）。
n_jobs：并行处理的作业数。设置为 -1 时使用所有可用的处理器核心。
总结来说，DBSCAN参数的选择主要围绕着如何准确地捕捉数据的密度分布来展开。合理调整eps和min_samples是成功应用DBSCAN的关键，这通常需要结合数据探索、可视化和实验验证。其他参数则根据数据特性和计算资源进行适当设置，以优化算法性能。
### KFold 交叉验证
sklearn.model_selection.KFold 是 scikit-learn 库中实现 K 折交叉验证的一种类。K 折交叉验证是一种常用的评估机器学习模型性能的方法，通过将数据集划分为 K 个大小相等（或尽可能接近相等）的子集（也称为“折”或“fold”），然后进行 K 次训练-验证循环，每次循环中用 K-1 个子集作为训练集，剩下的一个子集作为验证集。以下是 KFold 类的主要参数详解：

```python
sklearn.model_selection.KFold(n_splits=5, shuffle=False, random_state=None)
```
#### 参数:

 - n_splits: 整数，表示要将数据集划分为多少个折。默认为 5，即进行五折交叉验证。 
 - shuffle: 布尔值，指示是否在分割数据前对样本进行随机排序。如果为 True，则会对数据集进行随机打乱，以避免因原始数据顺序导致的偏差。默认为 False，即不进行随机打乱。
 - random_state: 可选，整数、RandomState 实例或 None。如果指定了整数，那么它会被用来作为 RandomState 的种子；如果是 RandomState 实例，则直接使用该实例进行随机数生成；如果为 None，则使用全局随机状态。设置此参数可以确保实验的可重复性。默认为 None。
#### 方法
`split(X, y=None, groups=None)`: 生成训练/验证集索引的迭代器。参数说明如下：
 - **X**: 输入数据，可以是 Numpy 数组、Pandas DataFrame 或任何类似数组的对象。
 - **y**: 可选，目标变量。如果没有提供，则假设数据集是无标签的。
 - **groups**: 可选，样本分组标识符。如果提供了groups，则会按照分组进行分层交叉验证，确保来自同一组的样本不会同时出现在训练集和验证集中。
#### 注意事项
 - **数据顺序与随机性**：当 shuffle=True 时，务必注意设置 random_state以确保实验可重复。如果不希望每次运行代码时得到不同的结果，请指定一个固定的种子值。
 - **分层交叉验证**：如果数据具有分组结构（如同一个个体的不同观测），应使用 groups 参数进行分层 K 折交叉验证，以保持组内样本的一致性。
 - **数据规模与 n_splits**：选择 n_splits 时应考虑数据集的大小。较大的 n_splits可以提供更稳定的模型性能估计，但会增加计算成本。较小的数据集可能不适合高 n_splits值，因为每个子集可能变得过小，导致模型训练不稳定。
## 参考
[https://xgboost.readthedocs.io/en/latest/](https://xgboost.readthedocs.io/en/latest/)
[https://lightgbm.readthedocs.io/en/latest/Parameters.html](https://lightgbm.readthedocs.io/en/latest/Parameters.html)
[https://catboost.ai/](https://catboost.ai/)
[https://tensorflow.google.cn/install?hl=zh-cn](https://tensorflow.google.cn/install?hl=zh-cn)
[https://keras.io/zh/](https://keras.io/zh/)
[https://scikit-learn.org/stable/modules/classes.html](https://scikit-learn.org/stable/modules/classes.html)
[https://datawhalechina.github.io/thorough-pytorch/](https://datawhalechina.github.io/thorough-pytorch/)

