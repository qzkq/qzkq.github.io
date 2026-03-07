---
title: 概率预测之NGBoost（Natural Gradient Boosting）回归和分位数（Quantile Regression）回归
date: 2026-01-05 08:00:00
tags:
  - "机器学习"
  - "NGBoost"
  - "概率预测"
  - "分位数回归"
  - "回归"
categories:
  - "机器学习"
---

﻿@[TOC](概率预测之NGBoost（Natural Gradient Boosting）回归和线性分位数回归)

概率预测是一种预测方法，它不仅提供一个具体的预测值（如点预测），还提供预测值的概率分布或置信区间。这种方法能够更好地捕捉预测的不确定性，适用于需要了解预测结果可靠性的场景。
# NGBoost
NGBoost（Natural Gradient Boosting）是一个用于提升树的分位数回归和概率预测的强大库。它通过自然梯度提升方法来优化分位数损失函数，从而能够提供更准确的概率预测和分位数回归。
## NGBoost超参数解释
1. n_estimators
含义：提升树的数量。
作用：控制模型的复杂度和拟合能力。增加树的数量可以提高模型性能，但也可能导致过拟合。
默认值：通常为50或100。
2. learning_rate
含义：学习率，用于缩放每棵树的贡献。
作用：降低每棵树的影响以防止过拟合，同时通过更多的树逐步逼近目标。
默认值：0.1。
3. minibatch_frac
含义：每次迭代时使用的样本比例（类似于随机梯度下降中的批量大小）。
作用：减少计算量并引入随机性，有助于防止过拟合。
默认值：1.0（使用所有样本）。
4. col_sample
含义：每次迭代时使用的特征比例。
作用：通过减少特征数量引入随机性，防止过拟合。
默认值：1.0（使用所有特征）。
5. base
含义：基础估计器（弱学习器），通常是决策树。
作用：指定模型的基础结构，默认为DecisionTreeRegressor。
默认值：max_depth=3 的决策树。
6. Dist
含义：目标变量的概率分布类型。
作用：定义目标变量的分布形式，如正态分布 (Normal)、伯努利分布 (Bernoulli) 等。
默认值：根据任务自动选择。
7. Score
含义：评分函数，用于评估当前模型的拟合效果。
作用：指导模型优化方向，例如负对数似然 (LogScore) 或偏差方差分解 (CRPScore)。
默认值：LogScore。
8. natural_gradient
含义：是否使用自然梯度下降。
作用：启用自然梯度下降可以加速收敛并减少训练过程中的振荡。
默认值：True。
9. verbose
含义：控制训练过程中的日志输出。
作用：调试和监控模型训练过程。
默认值：True。
10. random_state
含义：随机种子，确保结果可复现。
作用：设置随机数生成器的种子。
默认值：None。

```python
from xgboost import XGBRegressor

# 使用XGBoost作为基学习器
model = NGBRegressor(base=XGBRegressor(max_depth=3, n_estimators=10))

from sklearn.linear_model import LinearRegression

# 使用线性回归作为基学习器
model = NGBRegressor(base=LinearRegression())

from sklearn.svm import SVR

# 使用支持向量机作为基学习器
model = NGBRegressor(base=SVR(kernel='rbf'))
```
## NGBoost.fit 
 - X
类型：array-like 或 pandas.DataFrame
含义：训练数据的特征矩阵，形状为 (n_samples, n_features)。
作用：模型将基于这些特征进行学习。
 
 - Y
类型：array-like
含义：目标变量（标签），形状为 (n_samples,)。
作用：模型的目标是拟合这些标签值。
 
 - X_val=None
类型：array-like 或 pandas.DataFrame
含义：验证集的特征矩阵。
作用：如果提供验证集，则可以在训练过程中监控模型在验证集上的表现。
 
 - Y_val=None
类型：array-like
含义：验证集的目标变量。
作用：与 X_val 配合使用，用于评估模型的泛化能力。
 - early_stopping_rounds=None
类型：int
含义：早停轮数。
作用：如果在连续 early_stopping_rounds 轮中，验证集上的性能没有提升，则提前停止训练。用于防止过拟合。
 - sample_weight=None
类型：array-like
含义：样本权重，形状为 (n_samples,)。
作用：为每个样本分配不同的权重，影响模型的学习过程。

## score(X, Y)

 - 含义：计算模型在给定数据上的评分。
 - 返回值：负对数似然（Negative Log-Likelihood, NLL）或其他指定的评分函数值。

## staged_predict(X)

 - 含义：逐步生成预测结果，类似于梯度提升中的逐轮预测。
 - 参数：
	 - X：特征矩阵。
	 - 返回值：一个生成器，逐步返回每轮迭代后的预测结果。
	 - 适用场景：观察模型在不同迭代次数下的表现。
```python
for i, preds in enumerate(model.staged_predict(X_test)):
      print(f"第 {i+1} 轮预测: {preds[:5]}")
```
## feature_importances_
 - 含义：返回特征的重要性（基于基学习器的贡献）。
 - 返回值：一个数组，表示每个特征的重要性。
 - 注意：仅当基学习器为决策树时有效。

## pred_dist 方法来获取概率分布对象

 - mean(): 获取均值。
 - median(): 获取中位数。
 - interval(alpha): 获取指定置信水平的置信区间。
 - pdf(x): 获取概率密度函数（PDF）在 x 处的值。
 - cdf(x): 获取累积分布函数（CDF）在 x 处的值。
 - ppf(q): 获取分位数函数（PPF）在 q 处的值。

```python
import lightgbm as lgb
from sklearn.base import BaseEstimator, RegressorMixin
import numpy as np
from ngboost import NGBRegressor
from ngboost.scores import LogScore
from ngboost.distns import Normal
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error

# 定义适配器类
class LGBMRegressorAdapter(BaseEstimator, RegressorMixin):
    def __init__(self, **kwargs):
        self.lgbm = lgb.LGBMRegressor(**kwargs)
    
    def fit(self, X, y):
        self.lgbm.fit(X, y)
        return self
    
    def predict(self, X):
        return self.lgbm.predict(X)

# 生成模拟数据
X, Y = make_regression(n_samples=1000, n_features=10, noise=0.5, random_state=42)
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.2, random_state=42)

# 初始化 LGBMRegressorAdapter
lgbm_base = LGBMRegressorAdapter(n_estimators=10, max_depth=3, learning_rate=0.1)

# 初始化 NGBoost 模型，使用 LGBM 作为基学习器，并使用 LogScore 和 Normal
model = NGBRegressor(
    base=lgbm_base,
    Dist=Normal,
    Score=LogScore,
    n_estimators=200,
    learning_rate=0.1,
    verbose=True
)

# 训练模型
model.fit(
    X_train, Y_train,
    X_val=X_test, Y_val=Y_test,
    early_stopping_rounds=10,
    refit=True
)

# 预测
predictions = model.predict(X_test)

# 计算 MAE
mae = mean_absolute_error(Y_test, predictions)
print(f"Test MAE: {mae}")

# 获取概率分布对象
distributions = model.pred_dist(X_test)

# 示例：获取前5个样本的统计信息
for i, dist in enumerate(distributions[:5]):
    mean_val = dist.mean()
    median_val = dist.median()
    lower, upper = dist.interval(0.95)
    pdf_val = dist.pdf(mean_val)  # PDF at the mean
    cdf_val = dist.cdf(mean_val)  # CDF at the mean
    ppf_val = dist.ppf(0.5)       # PPF at 0.5 (median)
    
    print(f"样本 {i+1}:")
    print(f"  均值: {mean_val}")
    print(f"  中位数: {median_val}")
    print(f"  95% 置信区间: [{lower}, {upper}]")
    print(f"  PDF at mean: {pdf_val}")
    print(f"  CDF at mean: {cdf_val}")
    print(f"  PPF at 0.5: {ppf_val}")
    print()

# 以下等价
mean_val = dist.mean()
median_val = dist.median()
ppf_val = dist.ppf(0.5)       # PPF at 0.5 (median)
distributions = model.predict(X_test)
```
# 分位数回归（Quantile Regression）
分位数回归（Quantile Regression）是一种统计方法，用于估计目标变量在不同分位数上的条件分布。
##  smf.quantreg 对多变量数据进行分位数回归分析
smf.quantreg 是 statsmodels 库中的一个模块，用于进行分位数回归（Quantile Regression）。

```python
import pandas as pd
import statsmodels.formula.api as smf

# 创建示例数据集
data = {
    'y': [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    'x1': [10, 20, 30, 40, 50, 60, 70, 80, 90, 100],
    'x2': [15, 25, 35, 45, 55, 65, 75, 85, 95, 105]
}
df = pd.DataFrame(data)

# 指定分位数
quantile = 0.5

# 定义模型公式
formula = 'y ~ x1 + x2'

# 拟合模型
model = smf.quantreg(formula, df)
result = model.fit(q=quantile)

# 查看回归结果
print(result.summary())
```
## 概率预测指标

```python
# PICP: predicition interval coverage probability
# WS: winkler score
def evaluate_PICP_WS(y_pred_upper, y_pred_lower, y_test, confidence):
  
  # Reshape to 2D array for standardization
  y_pred_upper = np.reshape(y_pred_upper, (len(y_pred_upper), 1))
  y_pred_lower = np.reshape(y_pred_lower, (len(y_pred_lower), 1))
  
  y_pred_upper = sc_load.inverse_transform(y_pred_upper)
  y_pred_lower = sc_load.inverse_transform(y_pred_lower)
  y_test = sc_load.inverse_transform(y_test)
  
  # Ravel for ease of computation
  y_pred_upper = y_pred_upper.ravel()
  y_pred_lower = y_pred_lower.ravel()
  y_test = y_test.ravel()
  
  # Find out of bound indices for WS
  idx_oobl = np.where((y_test < y_pred_lower) > 0)
  idx_oobu = np.where((y_test > y_pred_upper) > 0)
  
  PICP = np.sum((y_test > y_pred_lower) & (y_test <= y_pred_upper)) / len(y_test) * 100
  WS = np.sum(np.sum(y_pred_upper - y_pred_lower) + 
              np.sum(2 * (y_pred_lower[idx_oobl[0]] - y_test[idx_oobl[0]]) / confidence) +
              np.sum(2 * (y_test[idx_oobu[0]] - y_pred_upper[idx_oobu[0]]) / confidence)) / len(y_test)
  
  print ("PICP of testing set: {:.2f}%".format(PICP))
  print ("WS of testing set: {:.2f}".format(WS))
  
  return PICP, WS
```

# 参考
1.[https://github.com/statsmodels/statsmodels](https://github.com/statsmodels/statsmodels)
2.新能源电力系统概率预测：基本概念与数学原理_万灿.pdf
3.新能源电力系统概率预测理论与方法=Theory and Methodology of Probabilistic Forecasting for Renewable Power Systems_万灿,宋永华.pdf
4.[https://github.com/stanfordmlgroup/ngboost](https://github.com/stanfordmlgroup/ngboost)


