---
title: XGBoost原生接口和Sklearn接口参数详解
date: 2026-01-03 08:19:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](XGBoost原生接口和Sklearn接口参数详解)
# [数据科学：Scipy、Scikit-Learn笔记](https://blog.csdn.net/qq_45832050/article/details/134815525)
# [超参数调优：网格搜索，贝叶斯优化（optuna）详解](https://blog.csdn.net/qq_45832050/article/details/138012101)
# [LightGBM原生接口和Sklearn接口参数详解](https://blog.csdn.net/qq_45832050/article/details/137434400)
# XGBoost
## 一、Sklearn风格接口
### xgboost.XGBRegressor参数

#### 一般参数
这些参数适用于 XGBoost 的核心算法，不特定于某个特定的弱学习器（如决策树）。
 1. max_depth (默认: 3)
类型: int
描述: 决策树的最大深度。限制树的生长高度，防止过拟合。值越大，模型可能更复杂，容易过拟合；值越小，模型可能欠拟合。
 2. learning_rate (默认: 0.1)
类型: float
描述: 学习率或步长。控制每次迭代中单个新树对最终模型影响的大小。较小的值（如 0.01 或 0.05）有助于降低过拟合风险，但可能需要更多的迭代次数（即更大的 n_estimators）。较大的值可以加速收敛，但可能导致模型不稳定或过拟合。
 3. n_estimators (默认: 100)
类型: int
描述: 弱学习器（树）的数量。增加此值会构建更多棵树，通常能提高模型的拟合能力，但也可能增加训练时间和模型复杂度。
 4. silent (默认: True)
类型: bool
描述: 是否禁用运行时信息输出。设为 False 可以显示训练过程中的进度信息。
 5. objective (默认: 'reg:squarederror')
类型: str
描述: 回归任务的目标函数。对于 XGBRegressor，默认设置为均方误差（Mean Squared Error, MSE），表示为 'reg:squarederror'。其他可用选项包括 'reg:linear'（线性回归）和 'reg:logistic'（逻辑回归，但通常用于二分类问题，此处不适用）。
 6. booster (默认: 'gbtree')
类型: str
描述: 使用的弱学习器类型。'gbtree' 表示基于树的提升（通常选择），另一种可选的是 'gblinear'，表示基于线性模型的提升。
 7. n_jobs (默认: 1)
类型: int
描述: 并行计算时使用的 CPU 核数。设置为 -1 则使用所有可用核。对于多核系统，增大此值可以加快训练速度。
 8. nthread (已弃用，用 n_jobs 替代)
类型: int
描述: 旧版本中用于指定线程数的参数，现已不再推荐使用，应改用 n_jobs。
弱评估器参数（Booster Parameters）
这些参数针对具体的弱学习器（如决策树）进行设置。
 9. gamma (默认: 0)
类型: float
描述: 分裂节点所需的最小损失减少量。较高的 gamma 值会限制树的复杂度，防止过拟合。
 10. min_child_weight (默认: 1)
类型: float
描述: 子节点所包含样本权重的最小和。较大的值可以防止模型学习到噪声或过于复杂的模式，避免过拟合。
#### 任务参数（Learning Task Parameters）
这些参数与特定的学习任务相关，如正则化、采样等。
 
 11. reg_alpha (默认: 0)
类型: float
描述: L1 正则化项的权重。非零值有助于简化模型，防止过拟合。
 13. reg_lambda (默认: 1)
类型: float
描述: L2 正则化项的权重。非零值有助于平滑模型，防止过拟合。
 14. subsample (默认: 1.0)
类型: float (0, 1]
描述: 训练过程中每个树构建时使用的样本比例。小于 1 的值可以引入随机性，有助于减少过拟合，相当于随机森林中的子采样。
 15. colsample_bytree (默认: 1.0)
类型: float (0, 1]
描述: 每棵树构建时考虑的特征比例。小于 1 的值可以引入特征的随机子集，有助于减少过拟合。
 16. colsample_bylevel (默认: 1.0)
类型: float (0, 1]
描述: 在树构建过程中各层级分裂时考虑的特征比例。类似于 colsample_bytree，但应用于每一层而不是每棵树。
 17. random_state (默认: None)
类型: int, RandomState instance, or None
描述: 随机数生成器种子或实例，用于确定随机性行为，如特征和样本的子采样。设置一个固定值可以确保实验的可复现性。
#### 其他重要参数
 17. eval_metric (默认取决于 objective)
类型: str, callable, list/tuple of str or list/tuple of callable
描述: 评估模型性能的度量标准。对于回归任务，常见的内置指标包括 'rmse'（均方根误差）、'mae'（平均绝对误差）等。
 19. early_stopping_rounds (默认: None)
类型: int
描述: 当验证集上的性能在一定轮数内未提升时，提前终止训练。配合 eval_set 和 eval_metric 使用，有助于防止过拟合并节省计算资源。
 20. verbose (默认: 0)
类型: int
描述: 控制训练过程中输出详细信息的程度。大于 0 的值将打印更多中间结果。

### XGBRegressor.fit参数
#### 基本参数
1. X (必需)
类型: array-like, DataFrame, DMatrix
描述: 训练数据，二维数组或类似结构，其中包含特征向量。数据类型可以是 NumPy 数组、Pandas DataFrame 或 XGBoost 自带的 DMatrix 数据结构。
2. y (必需)
类型: array-like, Series, DMatrix
描述: 目标变量，一维数组或类似结构，包含连续数值型的响应变量。数据类型可以是 NumPy 数组、Pandas Series 或 XGBoost DMatrix。
#### 可选参数
3. sample_weight (默认: None)
类型: array-like (shape (n_samples,))
描述: 样本权重。如果提供，将在训练过程中对每个样本赋予相应的权重，用于调整不同样本对模型学习的影响。权重越高，对应样本在训练中的作用越大。
4. eval_set (默认: None)
类型: list of (array-like, array-like) or list of DMatrix
描述: 用于模型评估的额外数据集列表，通常用于早停（early stopping）或监控训练过程中的性能。每个元素是一个元组，包含特征数据和相应的目标变量，格式与 (X, y) 相同。例如：[(X_val, y_val), (X_test, y_test)]。
5. eval_metric (默认: 取决于 objective)
类型: str, callable, list/tuple of str or list/tuple of callable
描述: 在训练过程中用于评估模型性能的度量标准。可以是一个字符串（代表内置评估指标），一个自定义函数，或它们的组合。对于回归任务，常用的内置指标包括 'rmse'（均方根误差）、'mae'（平均绝对误差）等。当提供了多个评估指标时，输出将按提供的顺序显示。
6. early_stopping_rounds (默认: None)
类型: int
描述: 当在 eval_set 上指定的评估指标在一定轮训练。这对于防止过拟合和数内未提升时，提前终止节省计算资源非常有用。例如，设置为 50 表示如果连续 50 轮后验证集上的性能没有提升，则停止训练。
7. verbose (默认: 0)
类型: int
描述: 控制训练过程中输出详细信息的程度。值越大，输出的信息越多。当设为 1 时，通常会显示每轮迭代后的评估指标；设为 0 则不显示。
8. callbacks (默认: None)
类型: list of callable objects
描述: 用户定义的回调函数列表，这些函数将在训练过程中特定时刻被调用。可用于实现自定义的日志记录、模型保存等行为。
9. base_margin (默认: None)
类型: array-like
描述: 用于初始化预测值（基线偏移）的一维数组。仅在高级用法中使用，通常情况下保持默认值即可。
10. sample_weight_eval_set (默认: None)
类型: list of arrays, description: 与 eval_set same length aseval_set- 中每个数据集对应的样本权重。若提供，将在评估这些数据集时使用对应的权重。
11. feature_weights (默认: None)
类型: array-like
描述: 特征的权重。在树构建过程中，可以用来调节不同特征的重要性。非零值会改变分裂节点时的增益计算方式。
12. categorical_feature (默认: None)
类型: list, int, dict
描述: 指定哪些特征是类别特征（离散特征）。可以是特征索引的列表、掩码数组，或字典（键为特征索引，值为类别数）。

### XGBRegressor.predict参数
#### 基本参数
 - data (必需)
类型: array-like, DataFrame, DMatrix
描述: 待预测数据，二维数组或类似结构，其中包含特征向量。数据类型可以是 NumPy 数组、Pandas DataFrame 或 XGBoost 自带的 DMatrix 数据结构。其列应与训练数据 X 的列相同，表示同样的特征。
可选参数
 - output_margin (默认: False)
类型: bool
描述: 如果设置为 True，将返回模型原始的未经转换的输出（边际）。对于回归任务，这通常是每个样本的预测值。如果设置为 False（默认），则返回经过目标函数转换后的预测值，如对于 'reg:squarederror' 目标函数，返回的是直接的预测值。
 - ntree_limit (默认: 0)
类型: int
描述: 限制用于预测的树的数量。若设为 0（默认），则使用所有已训练的树进行预测。非零值表示只使用前 ntree_limit 棵树进行预测。这个参数可用于探索模型在训练过程中不同阶段的表现，或者在早停后仅使用最优轮次的模型进行预测。
 - validate_features (默认: True)
类型: bool
描述: 检查预测数据 data 的特征是否与训练数据中的特征一致。如果设置为 True（默认），预测时会检查特征名称和类型是否匹配。如果特征不一致，会抛出异常。设置为 False 可以禁用此检查，但在实际应用中应谨慎使用，以避免因特征不匹配导致的预测错误。
 - base_margin (默认: None)
类型: array-like
描述: 用于初始化预测值（基线偏移）的一维数组。仅在高级用法中使用，通常情况下保持默认值即可。
 - iteration_range (默认: (0, num_boosted_rounds)，其中 num_boosted_rounds 为模型训练的总轮数)
类型: tuple (start, end)
描述: 限制用于预测的树的范围。(start, end) 表示从第 start 棵树开始，直到第 end - 1 棵树结束（包含 start，不包含 end）。这对分析模型在不同训练阶段的表现或仅使用部分树进行预测很有用。
#### 返回值
 - 类型: numpy.ndarray

描述: 返回一个一维 NumPy 数组，包含对 data 中每个样本的预测值。数组长度与 data 的行数相同。

## 二、XGBoost原生接口

```python
kfold = KFold(n_splits=5, random_state=42, shuffle=True)

mse = 0
for fold, (train_index, val_index) in enumerate(kfold.split(x, y)):
    logging.info(f'############ fold: {fold} ###########')
    x_train, x_val, y_train, y_val = x.iloc[train_index], x.iloc[val_index], y.iloc[train_index], y.iloc[val_index]
    
    trainset = Dataset(x_train, y_train)
    valset = Dataset(x_val, y_val)
    model = lgb.train(params_lgb, trainset, valid_sets=[trainset, valset], categorical_feature=["分"], callbacks=[lgb.log_evaluation(1000)])
    model.save_model("../models/lgb_%d.txt" % fold)
    model_lgb.append(model)
    lgb_pred = Series(model.predict(x_val, num_iteration=model.best_iteration), index=y_val.index).fillna(0)
    
    trainset = DMatrix(x_train, y_train, enable_categorical=True, nthread=-1)
    valset = DMatrix(x_val, y_val, enable_categorical=True, nthread=-1)
    model = xgb.train(params_xgb, trainset, evals=[(trainset, 'train'),(valset, 'eval')], num_boost_round=params_xgb["num_boost_round"], early_stopping_rounds=params_xgb["early_stopping_rounds"], verbose_eval=1000)
    model.save_model("../models/xgb_%d.json" % fold)
    model_xgb.append(model)
    xgb_pred = Series(model.predict(valset, iteration_range=(0, model.best_ntree_limit)), index=y_val.index).fillna(0)
    
    val_pred = (lgb_pred + xgb_pred) / 2
    mse += mean_squared_error(y_val.fillna(0), val_pred)
rmse = np.sqrt(mse / kfold.n_splits)
score = 1 / (1 + rmse)
logging.info(f"--------------本地分数 {score}--------------")
```

### 1. DMatrix 类
在使用 XGBoost 原生接口时，首先要将数据转换为 xgboost.DMatrix 对象，它是 XGBoost 库专门设计的数据结构，能够高效地存储和处理大量数据，特别是对于稀疏矩阵。

DMatrix 支持多种构造参数，如：

 - missing: 缺失值标记，默认为 NaN。可以指定其他缺失值标识符。 
 - weight: 样本权重。 base_margin: 基线偏移。
 -  label: 目标变量（仅在没有提供标签文件路径时需要）。
 - feature_names: 特征名称列表。
 -  feature_types: 特征类型列表，如 'q'（量化特征）、'i'（整数特征）、'f'（浮点特征）等。 
 - enable_categorical: 是否启用类别特征支持（仅在 XGBoost 1.6+ 版本中）。

### xgboost参数

 - **通用参数（General Parameters）**：影响整个梯度提升模型的构建和运行方式。
 - **Booster 参数（Booster Parameters）**：与特定的弱学习器（如决策树）有关，用于控制单个基础模型的构建细节。 
 - **任务参数（Learning Task Parameters）**：与特定的学习任务（如回归、分类）及正则化相关，影响模型在解决具体问题时的行为。
 -  **其他参数（Miscellaneous Parameters）**：包括模型输出、数据处理、并行计算等方面的设置。
#### 通用参数
**objective**

> 类型: str 
> 描述: 指定目标函数，决定了模型优化的目标。对于回归任务，常见的选项有 'reg:squarederror'（均方误差）、'reg:linear'（线性回归）、'reg:logistic'（逻辑回归，常用于二分类问题）等。

**booster**

> 类型: str 
> 描述: 指定使用的弱学习器类型。对于回归问题，通常选择 'gbtree'（基于树的提升），另一个选项是 'gblinear'（基于线性模型的提升）。

**verbosity**

> 类型: int 
> 描述: 控制日志输出的详细程度。值越大，输出的信息越多。通常设为 0 或 1。

**n_estimators / num_boost_round**

> 类型: int 
> 描述: 指定要构建的提升树（弱学习器）数量。增加该值通常可以提高模型的拟合能力，但可能会增加训练时间和过拟合风险。

**eta / learning_rate**

> 类型: float 
> 描述: 学习率或步长，控制每一步迭代中单个新树对最终模型影响的大小。较小的值可以减缓学习速度，有助于防止过拟合，但可能需要更多的迭代次数；较大的值可以加速收敛，但可能导致模型不稳定。

**gamma**

> 类型: float 
> 描述: 分裂节点所需的最小损失减少量。较高的值会限制树的复杂度，防止过拟合。

**max_depth**

> 类型: int 
> 描述: 决策树的最大深度。限制树的生长高度，防止过拟合。值越大，模型可能更复杂，容易过拟合；值越小，模型可能欠拟合。

**min_child_weight**

> 类型: float 
> 描述: 子节点所包含样本权重的最小和。较大值有助于防止模型学习到噪声或过于复杂的模式，避免过拟合。

**subsample / colsample_bytree**

> 类型: float (0, 1] 
> 描述: 分别控制训练样本和特征的子采样比例。小于 1
> 的值可以引入随机性，有助于减少过拟合，类似于随机森林中的子采样。

**lambda / reg_lambda / alpha / reg_alpha**

> 类型: float 
> 描述: L1 (reg_alpha) 和 L2 (reg_lambda) 正则化项的权重。非零值有助于简化模型，防止过拟合。

**n_jobs / nthread**

> 类型: int 
> 描述: 并行计算时使用的 CPU 核数。设置为 -1 则使用所有可用核。增大此值可以加快训练速度。

#### Booster 参数
**max_delta_step**

> 类型: float 
> 描述: 控制每棵树的权重更新幅度。对于某些数据集（如不平衡数据），有助于稳定模型训练。

**tree_method**

> 类型: str 
> 描述: 指定树构建算法。常见选项包括 'auto'（自动选择）、'exact'（精确贪心）、'approx'（近似贪心）、'hist'（基于直方图的构建）等。

**grow_policy**

> 类型: str 
> 描述: 决策树增长策略。 'depthwise'（深度优先）或 'lossguide'（损失导向）。

**monotone_constraints**

> 类型: str 或 list 
> 描述: 对特征施加单调性约束。例如，要求某个特征与目标变量的关系为单调递增或递减。

#### 任务参数
**scale_pos_weight**

> 类型: float 
> 描述: 仅对二分类任务有效，用于平衡正负类样本的权重。对于类别不平衡数据集，可以调整该参数来提高少数类样本的重要性。

**base_score**

> 类型: float 
> 描述: 初始预测分数（基线分数）。仅在使用非零基线时有意义。

**eval_metric**

> 类型: str, callable, list/tuple of str or list/tuple of callable 
> 描述: 指定评估模型性能的度量标准。内置指标包括 'rmse'、'mae'、'auc'、'logloss' 等。也可以自定义评估函数。

#### 其他参数
**random_state / seed**

> 类型: int, RandomState instance, or None 
> 描述: 随机数生成器种子或实例，用于确定随机性行为，如特征和样本的子采样。设置一个固定值可以确保实验的可复现性。

**missing**

> 类型: float 或 str 
> 描述: 缺失值标记。可以是浮点数或字符串（如 'nan'），用于指示数据中的缺失值。

**disable_default_eval_metric**

> 类型: bool 
> 描述: 是否禁用目标函数默认的评估指标。设为 True 可以仅使用自定义的 eval_metric。

### xgboost.train参数

1. params (必需)
类型: dict
描述: 包含训练参数的字典。这些参数会影响模型的构建和优化过程。常见参数包括：

2. dtrain (必需)
类型: xgboost.DMatrix
描述: 包含训练数据和标签的 DMatrix 对象。这是 XGBoost 专为高效处理大规模数据设计的数据结构。
3. evals (可选)
类型: list of (xgboost.DMatrix, str) tuples
描述: 用于模型评估和早停的数据集列表。每个元素是一个元组，包含特征数据和对应的目标变量，以及一个名称标识符。
4. eval_names (可选)
类型: list of str
描述: 与 evals 中数据集对应的名称列表。如果提供，将用于打印评估结果的标签。如果不提供，将使用 evals 中的第二元素作为名称。
5. obj (可选)
类型: callable
描述: 自定义目标函数。当内置目标函数无法满足需求时，可以通过此参数提供一个自定义的目标函数。函数签名应为 func(preds, dtrain)，其中 preds 是预测值，dtrain 是包含真实标签的 DMatrix。
6. feval (可选)
类型: callable
描述: 自定义评估函数。用于计算评估指标，特别是在内置评估指标不适用的情况下。函数签名应为 func(preds, dtrain)，其中 preds 是预测值，dtrain 是包含真实标签的 DMatrix。
7. early_stopping_rounds (可选)
类型: int
描述: 当在 evals 中指定的评估指标在一定轮数内未提升时，提前终止训练。这对于防止过拟合和节省计算资源非常有用。例如，设置为 50 表示如果连续 50 轮后验证集上的性能没有提升，则停止训练。
8. verbose_eval (可选)
类型: bool 或 int
描述: 控制训练过程中输出详细信息的程度。如果为 True 或正整数，将在每个验证周期输出评估结果。当设为正整数时，表示每隔这么多轮输出一次。设为 False 则不显示任何中间输出。
9. callbacks (可选)
类型: list of xgboost.callback.TrainingCallback instances
描述: 用户定义的回调函数列表，这些函数将在训练过程中特定时刻被调用。可用于实现自定义的日志记录、模型保存等行为。
10. custom_metric (可选)
类型: callable
描述: 自定义评估指标，用于替代 feval 参数。在 XGBoost 1.6 及更高版本中，推荐使用 custom_metric 替代 feval。

### xgboost.predict参数

 - data (必需)
类型: xgboost.DMatrix 或 numpy.array、pandas.DataFrame（在某些条件下）
描述: 待预测数据，必须与训练数据具有相同的特征空间。推荐使用 xgboost.DMatrix 类型，因为它针对 XGBoost 优化，具有更好的性能。也可以使用 numpy.array 或 pandas.DataFrame，但需确保它们与训练数据格式兼容，并且在必要时进行适当转换。
 - output_margin (可选)
类型: bool
描述: 控制是否返回模型的原始边际输出。默认为 False。设为 True 时，返回的是未经目标函数转换的预测值。对于回归任务，这通常是每个样本的预测值。设为 False（默认）时，返回经过目标函数转换后的预测值，如对于 'reg:squarederror' 目标函数，返回的是直接的预测值。
 - ntree_limit (可选)
类型: int
描述: 限制用于预测的树的数量。若设为 0（默认），则使用所有已训练的树进行预测。非零值表示只使用前 ntree_limit 棵树进行预测。这个参数可用于探索模型在训练过程中不同阶段的表现，或者在早停后仅使用最优轮次的模型进行预测。
 - validate_features (可选)
类型: bool
描述: 检查预测数据 data 的特征是否与训练数据中的特征一致。如果设置为 True（默认），预测时会检查特征名称和类型是否匹配。如果特征不一致，会抛出异常。设置为 False 可以禁用此检查，但在实际应用中应谨慎使用，以避免因特征不匹配导致的预测错误。
 - base_margin (可选)
类型: numpy.array
描述: 用于初始化预测值（基线偏移）的一维数组。仅在高级用法中使用，通常情况下保持默认值即可。
 - iteration_range (可选)
类型: tuple (start, end)
描述: 限制用于预测的树的范围。(start, end) 表示从第 start 棵树开始，直到第 end - 1 棵树结束（包含 start，不包含 end）。这对分析模型在不同训练阶段的表现或仅使用部分树进行预测很有用。
 - pred_leaf (可选)
类型: bool
描述: 控制是否返回每个样本落在各个决策树叶子节点的索引。设为 True 时，返回一个二维数组，形状为 (样本数, 树的数量)，每个元素表示对应样本在对应树中的叶子节点索引。默认为 False。
 - pred_contribs (可选)
类型: bool
描述: 控制是否返回每个特征对预测值的贡献。设为 True 时，返回一个三维数组，形状为 (样本数, 树的数量, 特征数)，每个元素表示对应样本在对应树中由对应特征带来的预测值增量。默认为 False。
 - approx_contribs (可选)
类型: bool
描述: （仅限二分类）控制是否使用近似方法计算特征贡献。仅在 pred_contribs=True 时生效。设为 True 可以加速计算，但精度可能降低。默认为 False。
#### 返回值
 - 类型: numpy.ndarray

描述: 返回一个一维或二维（取决于 pred_leaf 或 pred_contribs 是否启用）NumPy 数组，包含对 data 中每个样本的预测结果。数组形状和内容取决于所选参数。

### xgboost可视化
1.  xgboost.plot_importance
作用: 绘制特征重要性的条形图，展示每个特征对模型预测能力的相对贡献。
参数:

> booster: 已训练好的 XGBoost 模型对象。 importance_type: 特征重要性度量类型，可选 'weight'（基于节点分裂次数）、'gain'（基于节点增益）、'cover'（基于节点覆盖样本数）或 'total_gain'（总增益，仅限回归）。 
> 
> max_num_features: 显示的最大特征数（默认为 None，即显示所有特征）。
> 
> height: 图形的高度（默认为 0..png`，即创建新的图形）。
>  
> xlabel: X 轴标签（默认为 'Features'）。
> 
> ylabel: Y 轴标签（默认为 'Importance'）。 
> 
> title: 图形标题（默认为 'Feature Importance'）。 
> 
> ax: matplotlib.axes.Axes 对象，用于绘制图形。如果不提供，将创建一个新的图形。

2. xgboost.plot_split_value_histogram
作用: 绘制特征分裂阈值的直方图，帮助理解模型在构建决策树时如何分割特征。
参数:

> booster: 已训练好的 XGBoost 模型对象。 
> 
> feature_names: 特征名称列表，用于标签化 X 轴。
> 
> max_bins: 每个特征的最大分箱数（默认为 30）。 
> 
> ax: matplotlib.axes.Axes 对象，用于绘制图形。如果不提供，将创建一个新的图形。

3. xgboost.to_graphviz
作用: 将单个决策树转换为 Graphviz 格式，以便使用外部工具（如 graphviz 库或在线工具）进一步渲染成高质量的图形。
参数:

> booster: 已训练好的 XGBoost 模型对象。 
> 
> num_trees: 要转换的树的索引（默认为 0，即第一棵树）。
> 
> rankdir: 图形的方向，同 plot_tree 函数。 
> 
> hide_feature_names: 是否隐藏特征名称（默认为 False）。 
> 
> precision: 浮点数表示的精度（默认为 3）。

```python
import xgboost as xgb
import matplotlib.pyplot as plt

# 假设已有一个训练好的 XGBoost 模型：model

# 绘制第一个决策树
xgb.plot_tree(model, num_trees=0)
plt.show()

# 绘制特征重要性
xgb.plot_importance(model)
plt.show()

# 绘制特征分裂阈值直方图
# 注意：此处假设 `feature_names` 已知
xgb.plot_split_value_histogram(model, feature_names)
plt.show()

# 将第一个决策树转换为 Graphviz 格式
gviz = xgb.to_graphviz(model, num_trees=0)

# 使用 graphviz 库渲染图形（需先安装 graphviz）
from graphviz import Source
Source.from_file(gviz).view()
```
