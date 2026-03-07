---
title: Scikit-learn使用和扩展之mlxtend（Stacking...）
date: 2026-01-03 08:08:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](mlxtend)
**mlxtend**（Machine Learning Extensions）是一个Python库，它为Scikit-learn提供了额外的实用工具和扩展功能。mlxtend旨在为数据科学家和机器学习工程师提供一系列易于使用的高级API，以便于实现一些复杂的机器学习算法和技术，这些在标准的Scikit-learn库中可能没有直接提供。
## mlxtend的主要特点和功能

 - **Feature Selection**: 提供了多种特征选择算法，如基于顺序前进选择（Sequential Forward Selection）、顺序后退消除（Sequential Backward Elimination）等。
 - **Ensemble Methods**: 包括了如Stacking、Blending等集成学习方法，帮助用户构建更强大的集成模型。
 - **Model Evaluation**: 提供了如Bootstrap、Permutation Test等模型评估工具，用于更全面地评估模型性能。
 - **Classification and Regression Algorithms**: 实现了一些经典的学习算法，如Adaline、Perceptron等，以及一些更现代的算法，如Regularized Evolutionary Algorithms for Hyperparameter Optimization。
 - **Deep Learning**: 尽管主要关注于传统机器学习，mlxtend也提供了一些深度学习相关的功能，如自动编码器（Autoencoder）。
 - **Plotting Functions**: 提供了一系列绘图函数，用于可视化模型性能和数据分布，如混淆矩阵、决策区域等。
 - **Association Rules**: 实现了Apriori算法，用于挖掘交易数据中的频繁项集和关联规则。

mlxtend的设计理念是保持与Scikit-learn的高度兼容性和一致性，使得用户可以无缝地将mlxtend的功能整合到现有的Scikit-learn工作流中。这使得mlxtend成为了一个非常有用的工具箱，特别是在需要实现一些更高级的机器学习技术时。

## StackingCVRegressor详解
**StackingCVRegressor**是mlxtend库中用于实现Stacking集成学习的一种方法，尤其适用于回归问题。Stacking是一种高级集成学习技术，它通过训练一个元学习器（meta-learner）来结合多个基学习器（base learners）的预测结果，从而形成最终的预测。StackingCVRegressor通过交叉验证的方式来进行训练，这有助于减少过拟合的风险。
### 参数详解
**StackingCVRegressor的主要参数包括：**

 - **regressors**: 一个列表，包含了所有基学习器的实例。这些模型将被训练并用于生成元数据（meta-data）。
 - **meta_regressor**: 元学习器，用于结合基学习器的预测结果。这个模型将被训练来学习如何最优地结合基学习器的预测。
 - **cv**: 交叉验证的折数，默认是5。这决定了数据将被分割成多少份，以及元数据将如何被生成。
 - **shuffle**: 布尔值，决定是否在交叉验证前打乱数据集。默认是True。
 - **use_features_in_secondary**: 布尔值，决定元学习器是否除了使用基学习器的预测外，还使用原始特征进行训练。默认是False。如果设置为True，元学习器将得到一个包含基学习器预测和原始特征的更宽的数据集。
 - **store_train_meta_features**: 布尔值，决定是否存储训练集的元特征。这在调试或进一步分析时可能有用。默认是False。
 - **verbose**: 控制输出的详细程度。默认是0，表示没有输出。
 - **refit**: 布尔值，决定是否在交叉验证后使用整个数据集重新训练所有模型。默认是True。
 - **n_jobs**: 并行作业的数量。如果设置为-1，则使用所有可用的处理器。默认是1。
 - **pre_dispatch**: 控制并行作业的预调度数量。可以是一个整数或字符串'all'或'2*n_jobs'。
 - **random_state**: 控制随机状态的种子，对于可重复性很重要。

### 工作流程

 1. **基学习器训练**:    StackingCVRegressor将数据集分为cv个子集，然后对每个子集以外的数据训练基学习器。这样，每个基学习器都会产生对未参与训练的子集的预测。
 2.  **元数据生成**: 这些预测被用作元数据，即元学习器的输入特征。元数据通常包括所有基学习器对未参与训练的子集的预测。
 3. **元学习器训练**: 元学习器使用元数据和对应的真值标签进行训练，学习如何最优地结合基学习器的预测。
 4. **最终模型**: 如果refit参数为True，那么在交叉验证之后，所有基学习器和元学习器将使用整个数据集重新训练，以得到最终的Stacking模型。


```python
from mlxtend.regressor import StackingCVRegressor
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.linear_model import Ridge
from sklearn.datasets import load_boston
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error

# 加载数据
boston = load_boston()
X, y = boston.data, boston.target

# 划分数据
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 定义基学习器
regressors = [RandomForestRegressor(n_estimators=100, random_state=42),
              GradientBoostingRegressor(n_estimators=100, random_state=42),
              Ridge(alpha=1.0)]

# 定义元学习器
meta_regressor = Ridge(alpha=1.0)

# 创建StackingCVRegressor实例
stacking_regressor = StackingCVRegressor(regressors=regressors,
                                         meta_regressor=meta_regressor,
                                         cv=5,
                                         use_features_in_secondary=True)

# 训练模型
stacking_regressor.fit(X_train, y_train)

# 预测
y_pred = stacking_regressor.predict(X_test)

# 评估
mse = mean_squared_error(y_test, y_pred)
print(f'Mean Squared Error: {mse}')
```

## mlxtend.plotting.plot_learning_curves
在mlxtend库中，plot_learning_curves函数可以用来可视化模型的学习曲线，这对于理解模型在训练集和验证集上的性能随训练样本数量的变化情况非常有帮助。同时，当你使用Stacking集成学习方法时，plot_learning_curves可以帮助你评估Stacking模型的泛化能力和是否存在过拟合或欠拟合现象。

```python
mlxtend.plotting.plot_learning_curves(X_train, y_train, X_test, y_test, model, scoring=None, print_model=False, n_splits=5, n_train_sizes=10, style='ggplot', x_axis='train_sizes', ax=None, figsize=(10, 6), **kwargs)
```
### 参数说明

 - X_train: 训练集的特征数据。
 - y_train: 训练集的目标数据。
 - X_test: 测试集的特征数据。
 - y_test: 测试集的目标数据。
 - model: 要评估的模型，可以是任何具有fit和predict方法的对象。
 - scoring: 评估模型性能的指标，默认为None，表示使用模型的默认评分方法。
 - print_model: 是否在图中打印模型名称，默认为False。
 - n_splits: 交叉验证的折数，默认为5。
 - n_train_sizes: 训练集大小的样本数量，默认为10，表示在训练集大小的10个不同的点上计算性能。
 - style: 图表的样式，默认为'ggplot'，也可以选择'seaborn-whitegrid'等。
 - x_axis:   x轴的类型，可以是'train_sizes'（训练集大小）或'epochs'（迭代次数），默认为'train_sizes'。
 - ax: 可选的matplotlib轴对象，如果提供，则在该轴上绘制曲线。
 - figsize: 图形的大小，默认为(10, 6)。

```python
from mlxtend.plotting import plot_learning_curves
from mlxtend.regressor import StackingCVRegressor
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.linear_model import Ridge
from sklearn.datasets import load_boston
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt

# 加载数据
boston = load_boston()
X, y = boston.data, boston.target

# 划分数据
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 定义基学习器
regressors = [
    RandomForestRegressor(n_estimators=100, random_state=42),
    GradientBoostingRegressor(n_estimators=100, random_state=42),
    Ridge(alpha=1.0)
]

# 定义元学习器
meta_regressor = Ridge(alpha=1.0)

# 创建StackingCVRegressor实例
stacking_regressor = StackingCVRegressor(
    regressors=regressors,
    meta_regressor=meta_regressor,
    cv=5,
    use_features_in_secondary=True
)

# 训练Stacking模型
stacking_regressor.fit(X_train, y_train)

# 使用plot_learning_curves绘制学习曲线
train_scores, validation_scores = plot_learning_curves(
    X_train, y_train, X_test, y_test,
    stacking_regressor, scoring='mean_absolute_error',
    print_model=False,
    style='seaborn-whitegrid'
)

plt.title('Learning Curves (StackingCVRegressor)')
plt.xlabel('Training Set Size'), plt.ylabel('Neg Mean Squared Error Score'), plt.legend(loc="best")
plt.tight_layout()
plt.show()
```

## mlxtend.feature_selection.SequentialFeatureSelector
**SequentialFeatureSelector**是mlxtend库中用于特征选择的一种方法，它支持两种主要的特征选择策略：**顺序前进选择（Sequential Forward Selection, SFS）和顺序后向消除（Sequential Backward Selection, SBS）**。
**SequentialFeatureSelector 参数详解**
### estimator
类型：estimator object
描述：这是你想要用作特征选择的基础模型。它必须实现fit和predict方法，通常是一个分类器或回归器。
### k_features
类型：int or tuple
描述：指定要选择的特征数量。如果是整数，选择指定数量的特征。如果是tuple，例如(1, 5)，则会尝试从1到5的所有特征组合，并返回最佳的特征子集。
### forward
类型：bool
默认值：True
描述：如果True，则执行顺序前进选择（SFS），否则执行顺序后向消除（SBS）。
### floating
类型：bool
默认值：False
描述：如果True，则在每一步中除了添加或删除特征外，还会检查是否应该移除之前添加的特征（对于SFS）或者保留之前删除的特征（对于SBS）。这称为浮动选择。
### verbose
类型：int
默认值：0
描述：控制输出的详细程度。0表示无输出，1表示基本输出，2表示详细输出。
### scoring
类型：string, callable, list/tuple, dict or None
默认值：None
描述：用于评估模型性能的评分函数。可以是scikit-learn的评分字符串，如'accuracy'，'f1'，'roc_auc'等，也可以是自定义的评分函数。
### cv
类型：int, cross-validation generator or an iterable
默认值：5
描述：交叉验证的折数或生成器。如果是整数，则使用KFold交叉验证。
### n_jobs
类型：int or None
默认值：None
描述：并行作业的数量。如果设置为-1，则使用所有可用的处理器。
### pre_dispatch
类型：int or string
默认值：'2*n_jobs'
描述：控制并行作业的预调度。可以是整数或字符串，如'2*n_jobs'。
### clone_estimator
类型：bool
默认值：True
描述：是否在每次调用fit时克隆estimator。通常情况下，应该保持为True，除非你确定estimator在多次调用fit时能够正确地重置自己。
### fixed_features
类型：array-like, shape (n_fixed_features,)
默认值：None
描述：要始终包含在特征集合中的索引列表。如果None，则不包含任何固定特征。
### fixed_features_idx_
类型：array-like, shape (n_fixed_features,)
默认值：None
描述：与fixed_features相同，但在内部使用，不应由用户直接设置。
### features_
类型：array-like, shape (k_features,)
描述：选定的特征的索引数组。
### k_score_
类型：float
描述：使用选定特征获得的最佳评分。
### subsets_
类型：dict
描述：存储在特征选择过程中找到的所有子集的字典。
### k_feature_idx_
类型：tuple
描述：最佳特征子集的索引。
### k_feature_names_
类型：list
描述：如果输入特征有名称，这将是最佳特征子集的名称列表。

## mlxtend.plotting.plot_sequential_feature_selection

**plot_sequential_feature_selection**是mlxtend.plotting模块中的一个函数，用于可视化SequentialFeatureSelector的结果。这个函数提供了对特征选择过程中模型性能变化的图形展示，帮助理解和解释特征选择的效果。
**plot_sequential_feature_selection 参数详解**
### metric_dict
类型：dict
描述：这是SequentialFeatureSelector的get_metric_dict方法返回的字典，包含了在特征选择过程中收集的性能指标。这个字典是plot_sequential_feature_selection函数的主要输入，它包含了每一步特征选择的详细信息，包括选择的特征、评分等。
### kind
类型：str
默认值：'std_dev'
描述：确定绘制的误差线类型。可以是'std_dev'（标准偏差）、'std_err'（标准误差）或'ci'（置信区间）。这些选项决定了在性能评分的均值周围绘制的误差范围。
### ax
类型：matplotlib.axes.Axes object
默认值：None
描述：可选的Axes对象，用于在现有的图表上绘制学习曲线。如果未提供，将创建一个新的图表。
### figsize
类型：tuple
默认值：None
描述：如果ax未提供，可以使用figsize来设置新创建的图表的大小。
### grid
类型：bool
默认值：True
描述：是否在图表上显示网格线。
### xlabel
类型：str
默认值：'Number of Features'
描述：x轴的标签文本。
### ylabel
类型：str
默认值：'Score'
描述：y轴的标签文本。
### title
类型：str
默认值：None
描述：图表的标题。如果未提供，将使用默认标题。
### marker
类型：str
默认值：'o'
描述：在图表上使用的标记类型。
### markersize
类型：int
默认值：5
描述：标记的大小。
### linewidth
类型：int
默认值：2
描述：线条的宽度。
### alpha
类型：float
默认值：0.9
描述：线条和标记的透明度。
### ci_alpha
类型：float
默认值：0.15
描述：置信区间的透明度，仅当kind='ci'时有效。
### errorbar_kwargs
类型：dict
默认值：{}
描述：传递给matplotlib.errorbar的额外关键字参数，用于自定义误差线的样式。
### line_kwargs
类型：dict
默认值：{}
描述：传递给matplotlib.plot的额外关键字参数，用于自定义线条的样式。
### scatter_kwargs
类型：dict
默认值：{}
描述：传递给matplotlib.scatter的额外关键字参数，用于自定义散点图的样式。

```python
from mlxtend.plotting import plot_sequential_feature_selection
from mlxtend.feature_selection import SequentialFeatureSelector as SFS
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris
import matplotlib.pyplot as plt

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 创建模型和SFS对象
clf = LogisticRegression()
sfs = SFS(clf, k_features=3, forward=True, scoring='accuracy', cv=5)
sfs = sfs.fit(X, y)

# 获取metric字典
metric_dict = sfs.get_metric_dict()

# 绘制SFS结果
fig, ax = plt.subplots(figsize=(10, 6))
plot_sequential_feature_selection(metric_dict, kind='std_dev', ax=ax)
plt.title('Sequential Feature Selection (w. StdDev)')
plt.xlabel('Number of Features')
plt.ylabel('Accuracy')
plt.show()

```

## 函数
### feature_selection

> SequentialFeatureSelector: 顺序特征选择，包括前向选择和后向消除。
> ExhaustiveFeatureSelector: 穷举式特征选择，尝试所有可能的特征组合。
> SequentialFloatingFeatureSelector: 浮动顺序特征选择，结合了前向选择和后向消除。

### evaluate

> bootstrap: Bootstrap采样。 
> bias_variance_decomp: 偏差-方差分解。 
> learning_curve: 学习曲线。 
> scoring: 模型评分函数。 
> plot_learning_curves: 绘制学习曲线。
> permutation_test_score: Permutation测试评分。

### preprocessing

> MinMaxScaling: Min-Max缩放。 
> standardize: 标准化。 
> one_hot_encoding: One-hot编码。 
> transactionencoder: 交易数据编码。

### classifier

> Adaline: Adaline分类器。 
> Perceptron: 感知机分类器。 
> SoftmaxRegression: Softmax回归。
> LogisticRegression: 逻辑回归。 
> MultilayerPerceptron: 多层感知机。
> StackingClassifier: Stacking集成分类器。

### regressor

> LinearRegression: 线性回归。
>  RidgeRegression: 岭回归。 
>  LassoRegression: Lasso回归。 
>  ElasticNetRegression: 弹性网回归。 
>  StackingCVRegressor: Stacking集成回归器。

### plotting

> plot_decision_regions: 绘制决策区域。 
> plot_confusion_matrix: 绘制混淆矩阵。
> plot_sequential_feature_selection: 绘制顺序特征选择过程。

### association_rules

> apriori: Apriori算法，用于挖掘关联规则。 
> association_rules: 从频繁项集生成关联规则。

### genetic_algorithm

> GeneticAlgorithm: 基于遗传算法的特征选择。 
> neural_network: RecurrentNeuralNetwork: 循环神经网络。 
> Autoencoder: 自动编码器。

### sampling

> resample: 重采样。 
> smote: SMOTE过采样。

