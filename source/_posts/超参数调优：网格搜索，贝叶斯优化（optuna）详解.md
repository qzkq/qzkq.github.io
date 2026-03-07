---
title: 超参数调优：网格搜索，贝叶斯优化（optuna）详解
date: 2026-01-05 08:17:00
tags:
  - "机器学习"
  - "超参数调优"
  - "贝叶斯优化"
  - "网格搜索"
  - "Optuna"
categories:
  - "机器学习"
---

﻿@[TOC](超参数调优：网格搜索，贝叶斯优化（optuna）详解)

# [数据科学：Scipy、Scikit-Learn笔记](https://blog.csdn.net/qq_45832050/article/details/134815525)
# [LightGBM原生接口和Sklearn接口参数详解](https://blog.csdn.net/qq_45832050/article/details/137434400)
# [XGBoost原生接口和Sklearn接口参数详解](https://blog.csdn.net/qq_45832050/article/details/138009902)
# 网格搜索

GridSearchCV 是一个在 scikit-learn 库中用于执行网格搜索（grid search）参数调优的方法。网格搜索是一种通过遍历预定义的参数网格来确定机器学习模型最佳超参数组合的技术。它对给定的参数值集合中的所有可能组合进行训练和验证，最终选择具有最高交叉验证得分的参数配置。以下是对 GridSearchCV 类主要参数和属性的详细解析：
## 参数
### 1.estimator

 - 类型: estimator object

说明: 需要进行参数调优的基础学习器（模型）。这可以是任何实现了 fit() 方法的 scikit-learn estimator，如 LogisticRegression、SVM、RandomForestClassifier 或 GradientBoostingRegressor 等。
### 2.param_grid

 - 类型: dict or list of dictionaries

 说明: 指定要尝试的参数值的网格。它可以是一个字典，其中键是模型参数的名称，值是该参数可能取值的列表；也可以是一个字典列表，表示多个参数组合的网格搜索。例如：

```python
param_grid = {
    'parameter_name1': [value1, value2, ...],
    'parameter_name2': [valueA, valueB, ...],
    # 更多参数...
}
```
### 3.scoring

 - 类型: string, callable, list/tuple, dict or None, default=None

 说明: 用于评估模型性能的度量标准。它可以是内置评分字符串（如 'accuracy'、'roc_auc'、'neg_mean_squared_error' 等），自定义评分函数，或者对于多输出任务，可以是列表或字典形式的多个评分标准。若为 None，则使用 estimator 的默认评分方法。
### 4.fit_params

 - 类型: dict, optional

 说明: 可选的关键字参数传递给 estimator.fit() 方法。这些参数不会被网格搜索所改变，但可用于控制模型拟合过程，如设置正则化参数的随机种子 (random_state) 或指定特征重要性计算方式等。
### 5.n_jobs

 - 类型: int, default=1

说明: 并行处理数。若 -1，使用所有可用的CPU核心；若 1（默认），则顺序处理。大于 1 的整数表示使用相应数量的CPU核心。注意并行处理可能受到内存限制和其他因素的影响。
### 6.refit

 - 类型: bool, default=True

说明: 是否使用在交叉验证过程中找到的最佳参数重新拟合整个训练集。如果 True（默认），best_estimator_ 属性将包含使用最佳参数训练得到的模型。
### 7.cv

 - 类型: int, cross-validation generator, an iterable, or None,
   default=None

 说明: 交叉验证策略。可以是整数（代表折叠数，如 cv=5 表示五折交叉验证），特定的交叉验证生成器（如 KFold、StratifiedKFold），或者自定义的可迭代对象，产生（训练集，验证集）分割。若为 None，则使用 estimator 的默认交叉验证策略。
### 8.verbose

 - 类型: integer

 说明: 日志冗长度。控制输出信息的详细程度：

```python
0: 不输出训练过程信息。
1: 偶尔输出训练过程信息。
>1: 对每个子模型都输出训练过程信息。
```

### 9.return_train_score:

 - 类型: bool
 - 默认值: False

描述: 控制是否在网格搜索结果中包含训练得分。若设置为 True，将同时记录模型在训练集上的得分，以便进一步分析模型的过拟合或欠拟合情况。

 GridSearchCV.fit(X, y[, groups]) 方法是 sklearn.model_selection.GridSearchCV 类的一个重要方法，用于执行网格搜索过程，即遍历给定的参数网格，并针对每个参数组合利用交叉验证策略训练和评估模型。
调用 GridSearchCV.fit(X, y) 后，实例将保存以下属性，供后续分析和使用：

 - best_params_: 最佳参数组合，即在交叉验证过程中表现最好的参数设置。
 - best_estimator_: 使用最佳参数重新拟合得到的模型实例。
 - best_score_: 在交叉验证过程中，最佳参数组合对应的平均得分（基于指定的 scoring 函数）。
 - cv_results_: 字典形式的详细结果，包含了所有参数组合、得分、训练时间等信息。

## 属性
### 1.best_estimator_

 - 类型: estimator object

说明: 使用最佳参数组合训练得到的最优模型实例。仅当 refit=True 时有效。
### 2.best_score_

 - 类型: float

说明: 在交叉验证过程中观察到的最佳（平均）评分。
### 3.best_params_

 - 类型: dict

说明: 描述了获得最佳结果的参数组合，即字典形式的超参数及其对应的最优值。
### 4.best_index_

 - 类型: int

说明: 对应于最佳候选参数设置的索引，即 cv_results_ 数组中的索引位置。
### 5.cv_results_

 - 类型: dict of numpy (masked) ndarrays

说明: 包含了所有参数组合、交叉验证得分以及相关指标的详尽结果。这是一个丰富的字典结构，包含了关于每个参数组合的详细统计数据，如各个折叠得分、均值、标准差等。
# 贝叶斯优化（optuna）
**Optuna 是一个流行的Python库，专注于高效且直观地进行超参数优化。它旨在自动化机器学习（尤其是深度学习）模型的超参数搜索过程，以找到最优配置以提升模型性能。**
以下是对Optuna关键概念与参数的详细解析：
1. 研究（Study）
optuna.create_study(): 创建一个研究对象，它是超参数优化任务的容器。研究定义了优化的目标（即要最小化或最大化的指标）、搜索空间、以及优化算法（如随机搜索、TPE等）。

```python
study = optuna.create_study(
    study_name="example_study",
    direction="maximize",  # 或 "minimize"，根据目标函数的需求
    sampler=optuna.samplers.TPESampler(),  # 默认为RandomSampler，可指定其他采样器
)
```
2. 目标函数（Objective Function）
用户需提供一个目标函数，该函数接受一个包含待优化超参数的字典作为输入，并返回一个数值表示模型性能。Optuna会尝试不同的超参数组合，通过调用目标函数计算其性能，然后根据研究的方向（最大化或最小化）来指导搜索。

```python
def objective(trial):
    ...
    return accuracy  # 假设我们希望最大化accuracy
```
3. 超参数（Hyperparameters）
trial.suggest_*(): 在目标函数内部，使用Optuna提供的suggest方法来声明和获取超参数。这些方法包括不同类型的分布，如：
trial.suggest_float(): 定义一个连续浮点数范围。
trial.suggest_int(): 定义一个整数范围。
trial.suggest_categorical(): 定义离散的类别选择。
更多复杂类型，如trial.suggest_loguniform()、trial.suggest_discrete_uniform()等。

```python
def objective(trial):
    learning_rate = trial.suggest_float("learning_rate", 1e-5, 1e-2, log=True)
    num_layers = trial.suggest_int("num_layers", 1, 5)
    activation = trial.suggest_categorical("activation", ["relu", "sigmoid", "tanh"])
    ...
```
4. 采样器（Samplers）
optuna.samplers.*: 采样器决定了如何从搜索空间中抽取出超参数组合。Optuna提供了多种内置采样器，如：

> TPESampler: Tree-structured Parzen Estimator (TPE)，基于概率模型的适应性采样器，通常表现优秀。 
> RandomSampler: 随机搜索，简单但可靠。 
> GridSampler: 网格搜索，适用于小规模、离散参数空间。 
> CmaEsSampler: 基于Covariance Matrix Adaptation Evolution Strategy (CMA-ES)的进化算法。

选择采样器时应考虑搜索空间的特性、计算资源限制以及对探索与利用的平衡需求。

5. 优化（Optimization）
study.optimize(): 启动超参数优化过程。需要传入目标函数和相关参数，如最大试次数、时间限制等。

```python
study.optimize(objective, n_trials=100, timeout=600)  # 运行100次试验或最长600秒
```
6. 存储与重载（Storage & Reloading）
optuna.storages.*: Optuna支持将研究数据保存到各种后端存储，如SQLite、MySQL、Redis等，便于跨会话或分布式环境中的工作。

```python
study = optuna.create_study(
    storage="sqlite:///example.db",
    study_name="example_study",
    ...
)
```
之后可以使用相同的存储URL重新加载已存在的研究：

```python
study = optuna.load_study(study_name="example_study", storage="sqlite:///example.db")
```
7. 可视化与分析（Visualization & Analysis）
optuna.visualization.plot_*(): 提供了一系列图表来可视化研究结果，如参数重要性、优化历史、平行坐标图等。

```python
optuna.visualization.plot_param_importances(study)
optuna.visualization.plot_optimization_history(study)
```
Dashboard: Optuna还提供了一个交互式Web dashboard，可以实时监控优化过程和分析结果。启动dashboard：

```python
optuna-dashboard sqlite:///example.db --study example_study
```
8. 约束（Constraints）
可以通过在目标函数中添加条件判断来设置超参数组合的约束：

```python
def objective(trial):
    ...
    if learning_rate > 0.01 and batch_size < 32:
        raise optuna.structs.TrialPruned  # 若不符合条件，则标记该试验为被剪枝
```
9. 早停（Early Stopping）
optuna.trial.Trial.report() 和 optuna.trial.Trial.should_prune(): 可以在目标函数中报告中间结果，并检查是否应提前终止当前试验（早停）。这有助于节省计算资源。

```python
def objective(trial):
    for epoch in range(num_epochs):
        ...
        intermediate_value = val_loss
        trial.report(intermediate_value, step=epoch)
        if trial.should_prune(epoch):
            raise optuna.structs.TrialPruned
```
10. 多目标优化（Multi-objective Optimization）
optuna.create_study(multivariate=True): 支持多目标优化，目标函数返回一个包含多个目标值的列表。

```python
import optuna

def objective(trial):
    lgb_params = {
        "verbosity": -1,
        'objective': 'regression',
        'metric': 'rmse',
        'boosting_type': 'gbdt',
        'random_state': 6,
        'n_estimators': trial.suggest_int('n_estimators', 50, 200),
        'reg_alpha': trial.suggest_loguniform('reg_alpha', 1e-3, 10.0),
        'reg_lambda': trial.suggest_loguniform('reg_lambda', 1e-3, 10.0),#对数分布的建议值
        'colsample_bytree': trial.suggest_float('colsample_bytree', 0.5, 1),#浮点数
        'subsample': trial.suggest_float('subsample', 0.5, 1),
        'learning_rate': trial.suggest_float('learning_rate', 1e-4, 0.5, log=True),
        'num_leaves' : trial.suggest_int('num_leaves', 8, 64),#整数
        'min_child_samples': trial.suggest_int('min_child_samples', 1, 100),
    }
    X=train_feats.drop(['target'],axis=1).copy()
    y=train_feats['target'].copy()
    test_X=valid_feats.drop(['target'],axis=1).values.copy()
    test_y=valid_feats['target'].values.copy()
    test_preds=np.zeros((5,len(test_X)))
    # 初始化 KFold
    kf = KFold(n_splits=5, shuffle=True,random_state=6)
    # 进行 k 折交叉验证
    for fold, (train_index, valid_index) in (enumerate(kf.split(X))):
        X_train, X_valid = X.iloc[train_index], X.iloc[valid_index]
        y_train, y_valid = y.iloc[train_index], y.iloc[valid_index]

        model=LGBMRegressor(**lgb_params)
        model.fit(X_train,y_train)
        test_preds[fold]=model.predict(test_X)
    test_preds=test_preds.mean(axis=0)
    mean_rmse=metric(test_y,test_preds)
    return mean_rmse
#创建的研究命名,找最小值.
study = optuna.create_study(direction='minimize', study_name='Optimize boosting hyperparameters')
#目标函数,尝试的次数
study.optimize(objective, n_trials=50)
lgb_params=study.best_trial.params
```


