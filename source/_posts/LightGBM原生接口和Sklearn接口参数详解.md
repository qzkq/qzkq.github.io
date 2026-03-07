---
title: LightGBM原生接口和Sklearn接口参数详解
date: 2026-01-01 08:13:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](LightGBM原生接口和Sklearn接口参数详解)
# [数据科学：Scipy、Scikit-Learn笔记](https://blog.csdn.net/qq_45832050/article/details/134815525)
# [超参数调优：网格搜索，贝叶斯优化（optuna）详解](https://blog.csdn.net/qq_45832050/article/details/138012101)
# [XGBoost原生接口和Sklearn接口参数详解](https://blog.csdn.net/qq_45832050/article/details/138009902)
# LightGBM
**LightGBM有Sklearn接口建模和原生建模两种方式。**
## 一、Sklearn风格接口

```python
import numpy as np
import pandas as pd
from lightgbm import LGBMRegressor
# 使用普通的k折交叉验证
from sklearn.model_selection import KFold

lgb_params= {'objective': 'regression', 'metric': 'rmse',
             'boosting_type': 'gbdt', 'random_state': 2024}
             
test_preds=np.zeros((5, len(test_X)))
# 初始化 KFold
kf = KFold(n_splits=5, shuffle=True, random_state=6)
# 进行 k 折交叉验证
for fold, (train_index, valid_index) in (enumerate(kf.split(X))):

    print(f"fold:{fold}")

    X_train, X_valid = X.iloc[train_index], X.iloc[valid_index]
    y_train, y_valid = y.iloc[train_index], y.iloc[valid_index]

    model=LGBMRegressor(**lgb_params)
    model.fit(X_train,y_train,eval_set=[(X_valid,y_valid)])
    test_preds[fold]=model.predict(test_X)
```

### lightgbm.LGBMRegressor参数
**基本参数**
#### 1.boosting_type:

 - 默认值: 'gbdt'
 - 可选值: 'gbdt', 'dart', 'goss', 'rf'

> 'gbdt': 常规梯度提升决策树 (Gradient Boosting Decision Tree) 
> 'dart':引入Dropout的随机梯度提升 (Dropouts meet Multiple Additive Regression Trees)
> 'goss': 平衡梯度提升 (Gradient-based One-Side Sampling) 
> 'rf': 随机森林 (Random Forest)，实际上每个树都是独立训练的，而非传统的梯度提升方式

#### 2.objective:

 - 类型: str 或 callable
 - 默认值: 'regression'

说明: 定义优化目标函数。对于回归任务，通常保持默认值即可。如果有特定的回归损失需求（如 Huber loss），可以指定相应的字符串或自定义损失函数。
#### 3.metric:

 - 类型: str, list[str], callable

说明: 评估模型性能的指标。对于回归任务，常用的有 'l1', 'l2', 'rmse', 'mae', 'rmsle', 'mape' 等。可以指定多个指标，但最终模型会在训练过程中优化第一个指定的指标。
#### 4.n_estimators:

 - 类型: int
 - 默认值: 100

说明: 决策树的个数（即迭代次数）。增大此参数通常会提高模型复杂度和拟合程度，但可能导致过拟合。需要通过交叉验证或其他方法寻找合适的值。
#### 5.learning_rate (eta):

 - 类型: float
 - 默认值: 0.1

说明: 学习率，控制每一步迭代中单个新树对之前模型更新的幅度。较小的学习率可能需要更多的迭代次数（更大的 n_estimators），但有助于模型收敛到更优解并防止过拟合。通常在 [0.01, 0.2] 范围内调整。
#### 6.max_depth:

 - 类型: int
 - 默认值: -1（无限制）

说明: 树的最大深度。限制树的深度有助于防止过拟合并加快训练速度，但过小可能导致模型欠拟合。根据数据特性和问题复杂性调整。
#### 7.min_child_samples:

 - 类型: int
 - 默认值: 20

说明: 构建子树时，内部节点所需最小样本数。用于剪枝和控制模型复杂度。

**正则化与早停参数**
#### 8.lambda_l1 (reg_alpha):

 - 类型: float
 - 默认值: 0

说明: L1 正则化项的权重，用于控制模型的稀疏性。非零值有助于特征选择。
#### 9.lambda_l2 (reg_lambda):

 - 类型: float
 - 默认值: 0

说明: L2 正则化项的权重，用于控制模型的复杂度和防止过拟合。通常与 lambda_l1 结合使用。
#### 10.num_leaves:

 - 类型: int
 - 默认值: 31

说明: 树上叶子节点的最大数量。直接影响模型复杂度，与 max_depth 一起控制模型精细化程度。
#### 11.min_split_gain:

 - 类型: float
 - 默认值: 0

说明: 分裂内部节点所需的最小增益。用于避免不必要的分割。
#### 12.early_stopping_rounds:

 - 类型: int
 - 默认值: None

说明: 若在验证集上连续若干轮（指定值）未见性能提升，则提前停止训练，防止过拟合并节省计算资源。

**其他参数**

#### 13.subsample_for_bin:

 - 类型: int
 - 默认值: 200000

说明: 用于构建直方图的样本数量。影响特征离散化的效率和质量。
#### 14.colsample_bytree (feature_fraction):

 - 类型: float
 - 默认值: 1.0

说明: 每棵树构建时使用的特征比例。小于 1.0 可以增加模型多样性，防止过拟合。
#### 15.subsample (bagging_fraction):

 - 类型: float
 - 默认值: 1.0

说明: 训练每棵树时使用的样本比例。类似于随机森林中的采样策略，可以降低方差并加速训练。
#### 16.n_jobs:

 - 类型: int
 - 默认值: -1 (使用所有可用CPU核心)

说明: 并行计算的进程数。正值表示具体进程数，-1 表示使用所有可用 CPU 核心。
#### 17.silent (verbose):

 - 类型: bool
 - 默认值: True

说明: 控制是否输出详细的中间信息。在调试或监控训练过程时可设为 False。

### 调参方法

通常采用**网格搜索**（如 **GridSearchCV**）、随机搜索（如 RandomizedSearchCV）或**贝叶斯优化**（如 **Optuna**、Hyperopt）等方法来寻找 LGBMRegressor 的最佳参数组合。结合交叉验证评估不同参数配置下的模型性能，依据选定的评估指标（如 r2_score、mean_squared_error 等）确定最优参数。
注意，参数之间可能存在交互效应，如 **learning_rate 和 n_estimators 通常需同时调整**。**实践中应优先关注对模型性能影响显著的参数（如 learning_rate、n_estimators、num_leaves 等），然后逐步细化其他参数。同时，合理设置参数搜索范围和步长，确保搜索效率与精度之间的平衡。**

### LGBMRegressor.fit参数
LGBMRegressor.fit 是 LightGBM 回归模型训练方法，用于拟合给定的训练数据。以下是 LGBMRegressor.fit 方法的主要参数详解：
**必填参数**
#### 1.X:

 - 类型: array-like 或 DataFrame

说明: 输入特征数据，形状为 (n_samples, n_features)。这里的 n_samples 是样本数量，n_features 是特征数量。数据类型可以是整型、浮点型或其他可转换为数值的数据类型。
#### 2.y:

 - 类型: array-like 或 Series

说明: 目标变量（响应变量）数据，形状为 (n_samples,)。对于回归任务，y 应包含连续数值型标签。

**可选参数**

#### 3.sample_weight:

 - 类型: array-like of shape (n_samples,), optional
 - 默认值: None

说明: 样本权重数组，与输入数据 X 的样本一一对应。如果提供了样本权重，LightGBM 将在训练过程中对每个样本赋予相应的权重，影响其对模型拟合的贡献。权重越大，相应样本在训练时的影响力越强。这对于处理不平衡数据或对某些样本给予更多重视的情况非常有用。
#### 4.eval_set:

 - 类型: list of (X, y) tuple(s), optional
 - 默认值: None

说明: 用于评估模型性能的额外数据集列表。每个元素是一个包含特征数据 X 和目标变量 y 的元组。在训练过程中，LightGBM 会在每个 boosting 步骤后计算指定数据集上的评估指标，便于监控模型性能和进行早停。
#### 5.eval_names:

 - 类型: list of str, optional
 - 默认值: None

说明: 与 eval_set 对应的评估数据集名称列表。如果不提供，将自动命名为 "valid_0", "valid_1", ...。这些名称用于输出和日志中标识不同的评估数据集。
#### 6.eval_sample_weight:

 - 类型: list of array-like, optional
 - 默认值: None

说明: 与 eval_set 对应的评估数据集样本权重列表。如果提供了样本权重，将在评估过程中对相应数据集的样本应用这些权重。
#### 7.eval_class_weight:

 - 类型: dict or list of dict, optional
 - 默认值: None

说明: 仅适用于分类任务（与 LGBMRegressor 不相关），用于指定评估数据集中各类别的权重。
#### 8.eval_init_score:

 - 类型: list of array-like, optional
 - 默认值: None

说明: 用于初始化模型在每个评估数据集上的初始预测分数。如果提供了，将在第一轮迭代前使用这些分数作为起始点。
#### 9.eval_metric:

 - 类型: str, callable, list, optional
 - 默认值: None

说明: 在训练过程中使用的评估指标，可以是 LightGBM 支持的字符串标识符（如 'rmse', 'l1', 'mae' 等），也可以是自定义的评估函数。若未指定，将使用模型初始化时设定的 metric 参数。若提供列表，将在每个评估数据集上分别计算多个指标。
#### 10.verbose:

 - 类型: int or bool
 - 默认值: None

说明: 控制训练过程中的输出详细程度。如果为 True 或非零整数，将在标准输出流显示训练进度。整数值决定了信息刷新的频率（按迭代次数计）。若为 False 或 0，则不显示任何训练信息。
#### 11.callbacks:

 - 类型: list of callable, optional
 - 默认值: None

说明: 用户自定义的回调函数列表，在训练过程中的特定时刻（如每个 boosting 步骤后）会被调用。回调函数可以用于监控训练进度、保存中间结果、执行自定义操作等。
#### 12.init_model:

 - 类型: str, os.PathLike object, Booster instance, or dict, optional
 - 默认值: None

说明: 用于加载已有的 LightGBM 模型作为训练起点。可以是模型文件路径、 Booster 实例或序列化后的字典。加载模型后，训练将继续在现有模型基础上进行，而不是从头开始。
#### 13.pre_partition:

 - 类型: bool
 - 默认值: False

说明: 当设置为 True 时，表示数据已经按照 LightGBM 的要求进行了预分区。这通常在分布式训练场景中使用，以避免重复分区带来的开销。

**其他参数**

LGBMRegressor.fit 方法还接受 LGBMRegressor 类实例化时设置的其他参数作为关键字参数，如 num_boost_round、early_stopping_rounds、verbose_eval 等。这些参数在模型训练过程中起作用，可以通过直接在 fit 方法中传递来覆盖实例化时的设置。
总之，LGBMRegressor.fit 方法允许用户指定训练数据、权重、评估数据集、评估指标、输出详细程度等多个方面，以灵活控制模型训练过程和监控模型性能。在实际应用中，可以根据项目需求选择性地使用这些参数。
### LGBMRegressor.predict参数

LGBMRegressor.predict() 是 LightGBM 回归模型用于对新数据进行预测的方法。

```python
LGBMRegressor.predict(X, num_iteration=None, raw_score=False, pred_leaf=False, pred_contrib=False, **kwargs)
```

#### 1. X

 - 类型: array-like 或 DataFrame

说明: 待预测的数据，其形状应与训练数据的特征部分（不包含标签）相同。可以是 Numpy 数组、Pandas DataFrame 或类似的数据结构，列对应特征，行对应样本。
#### 2. num_iteration (n_iter_no_change)

 - 类型: int 或 None
 - 默认值: None

说明: 使用多少个树（迭代次数）来进行预测。如果为 None，则使用全部已训练好的树进行预测。指定一个较小的值可以观察模型在早期迭代阶段的表现，有助于理解模型的学习过程。
#### 3. raw_score

 - 类型: bool
 - 默认值: False

说明: 是否返回原始得分（即未经过学习目标函数转换的输出）。若设为 True，返回的是每个样本在最后一层叶节点的得分（通常是加权投票或平均），而不是经过目标函数转换后的预测值。对于回归任务，原始得分通常就是叶节点的均值。
#### 4. pred_leaf

 - 类型: bool
 - 默认值: False

说明: 是否返回每个样本在每棵树上的叶节点索引。若设为 True，返回的是一个二维数组，形状为 (n_samples, n_trees)，其中每个元素表示对应的样本在对应树中的叶节点编号。这个功能常用于模型解释和可视化。
#### 5. pred_contrib

 - 类型: bool
 - 默认值: False

说明: 是否返回特征对预测结果的边际贡献（SHAP-like values）。仅在模型训练时设置了 enable_pred_contrib=True 才有效。若设为 True，返回的是一个三维数组，形状为 (n_samples, n_features, n_classes)，其中每个元素表示对应样本、对应特征对预测结果的边际贡献。对于回归任务，n_classes=1。
#### 6. kwargs

 - 类型: 其他关键字参数

说明: 传递给底层 Booster.predict() 函数的额外参数。一般情况下不需要设置。
返回值:

如果没有启用 raw_score、pred_leaf 和 pred_contrib，返回一个一维 Numpy 数组，包含对 X 中每个样本的预测值。

如果启用了上述任一选项，返回相应类型的数组或数据结构，如原始得分、叶节点索引或特征边际贡献。

总结来说，LGBMRegressor.predict() 主要接收待预测数据 X 以及几个控制预测输出特性的参数，如指定使用多少棵树进行预测、是否返回原始得分、叶节点索引或特征边际贡献等。根据实际需求选择合适的参数，可以获得模型在不同层面的预测结果，以满足分析、解释或应用的需求。

## 二、LightGBM原生接口
LightGBM内置了建模方式，有如下的数据格式与核心训练方法：

 - 基于lightgbm.Dataset格式的数据。
 - 基于lightgbm.train接口训练。

```python
import numpy as np
import pandas as pd
import lightgbm as lgb
from sklearn.model_selection import KFold

def cv_model(clf, train_x, train_y, test_x, seed=2024):
    folds = 5
    kf = KFold(n_splits=folds, shuffle=True, random_state=seed)
    oof = np.zeros(train_x.shape[0])
    test_predict = np.zeros(test_x.shape[0])
    cv_scores = []
    
    for i, (train_index, valid_index) in enumerate(kf.split(train_x, train_y)):
        print('**************** {} ****************'.format(str(i+1)))
        trn_x, trn_y, val_x, val_y = train_x.iloc[train_index], train_y[train_index], train_x.iloc[valid_index], train_y[valid_index]
        
        train_matrix = clf.Dataset(trn_x, label=trn_y)
        valid_matrix = clf.Dataset(val_x, label=val_y)
        params = {
            'boosting_type': 'gbdt',
            'objective': 'regression',
            'metric': 'rmse',
            'seed': 2023,
            'nthread' : 16,
            'verbose' : -1,
        }

        model = clf.train(params, train_matrix, valid_sets=[train_matrix, valid_matrix], verbose_eval=500, early_stopping_rounds=200)
        val_pred = model.predict(val_x, num_iteration=model.best_iteration)
        test_pred = model.predict(test_x, num_iteration=model.best_iteration)
        
        oof[valid_index] = val_pred
        test_predict += test_pred / kf.n_splits
        
        score = 1/(1+np.sqrt(mean_squared_error(val_pred, val_y)))
        cv_scores.append(score)
        print(cv_scores)
        
        if i == 0:
            imp_df = pd.DataFrame()
            imp_df["feature"] = cols
            imp_df["importance_gain"] = model.feature_importance(importance_type='gain')
            imp_df["importance_split"] = model.feature_importance(importance_type='split')
            imp_df["mul"] = imp_df["importance_gain"]*imp_df["importance_split"]
            imp_df = imp_df.sort_values(by='mul',ascending=False)
            imp_df.to_csv('feature_importance.csv', index=False)
            print(imp_df[:30])
            
    return oof, test_predict

lgb_oof, lgb_test = cv_model(lgb, train_df[cols], train_df['power'], test_df[cols])

```
LightGBM 是一个高效的梯度提升决策树（GBDT）框架，支持分类和回归任务。其参数众多，可以细分为以下几大类别：
### 基本设置
#### 1.boosting_type: 
梯度提升类型。可选值包括 'gbdt'（传统梯度提升决策树）、'dart'（Dropouts meet Multiple Additive Regression Trees）、'goss'（Gradient-based One-Side Sampling）等。默认为 'gbdt'。
#### 2.objective: 
目标函数。对于回归任务，常用值包括 'regression_l1'（L1 loss）、'regression_l2'（L2 loss，即均方误差）、'huber'（Huber loss）等。默认为 'regression'（等价于 'regression_l2'）。
#### 3.metric: 
评估指标。可选值包括 'l1'、'l2'、'rmse'、'mae'、'rmsle' 等。可以指定多个评估指标，主指标用于早停和最优模型的选择。
### 数据处理与采样
#### 4.max_bin: 
构建直方图的最大bins数，影响特征离散化的精细程度。默认为 255。
#### 5.min_data_in_bin: 
一个bin中最小的数据数量。默认为 5。
#### 6.min_child_samples: 
创建子节点所需的最小样本数。默认为 20。
#### 7.subsample: 
数据采样的比例。取值范围为 (0, 1)，默认为 1 表示使用全部数据。
#### 8.sampling_method: 
采样方法。可选值包括 'uniform'（均匀采样）和 'gradient_based'（基于梯度采样）。默认为 'uniform'。
#### 9.colsample_bytree: 
每棵树随机选择特征的比例。取值范围为 (0, 1)，默认为 1 表示使用所有特征。
#### 10.colsample_bylevel: 
在每层节点分裂时使用的特征采样比例。取值范围为 (0, 1)，默认为 1。
### 模型训练与正则化
#### 11.learning_rate: 
学习率，控制每次迭代更新权重的步长。默认为 0.1。
#### 12.num_leaves: 
每棵树的最大叶子节点数。影响模型复杂度。默认为 31。
#### 13.min_split_gain: 
创建新节点所需的最小增益。用于防止过拟合。默认为 0。
#### 14.min_child_weight: 
创建子节点所需的最小权重（样本数乘以样本权重）。默认为 1e-3。
#### 15.lambda_l1/lambda_l2: 
L1 和 L2 正则化项的系数，分别用于惩罚绝对值和平方和。默认均为 0。
#### 16.alpha: 
Dart 算法中的 dropout 参数。默认为 0。
#### 17.n_estimators/num_iterations: 
最大迭代次数（树的数量）。默认为 -1，表示无限制。
#### 18.early_stopping_rounds: 
早停轮数。当验证集分数在一定轮数内不再提升时停止训练。默认为 None。
### 树结构与剪枝
#### 19.max_depth: 
模型最大深度。若设置为 -1，表示不限制深度。默认为 -1。
#### 20.tree_learner: 
树学习器类型。可选值包括 'serial'、'feature_parallel'、'data_parallel' 和 'voting'。默认为 'serial'。
#### 21.verbosity: 
输出日志的详细程度。默认为 1。
#### 22.gamma: 
节点分裂的最小损失减少。默认为 0。
#### 23.grow_policy: 
决策树增长策略。可选值包括 'depthwise'（深度优先）和 'lossguide'（损失降低优先）。默认为 'depthwise'。
分布式训练
#### 24.num_threads/n_jobs: 
使用的线程数。默认为 -1，表示使用所有可用CPU核心。
#### 25.distribute: 是否开启分布式训练。默认为 False。
相关参数（如 machine_list_file、local_listen_port 等）：用于配置分布式训练环境。
### 其他
#### 26.seed: 
随机种子，用于确保模型训练的可复现性。默认为 None。
#### 27.callbacks: 
回调函数列表，用于监控训练过程并在特定事件发生时执行操作。

### lightgbm.train参数

```python
lightgbm.train(params, train_set, num_boost_round=100, valid_sets=None, valid_names=None,
               fobj=None, feval=None, init_model=None, feature_name='auto', categorical_feature='auto',
               early_stopping_rounds=None, evals_result=None, verbose_eval=True, learning_rates=None,
               keep_training_booster=False, callbacks=None, show_stdv=True)
```
#### 1.params: 
字典类型，包含模型训练的参数，如 objective、metric、num_leaves、learning_rate 等。
#### 2.train_set: 
lightgbm.Dataset 类型，表示训练数据集。
#### 3.num_boost_round: 
整数，指定最大迭代次数（树的数量）。默认为 100。
#### 4.valid_sets: 
可选，lightgbm.Dataset 类型的列表，表示验证数据集。可以包含多个数据集，用于监控模型在不同数据上的表现。
#### 5.valid_names: 
可选，与 valid_sets 配对的字符串列表，为每个验证数据集指定名称。
#### 6.fobj: 
可选，自定义目标函数（Objective Function），用于替换默认的目标函数。需返回一个 (grad, hess) 元组，分别代表梯度和二阶导数。
#### 7.feval: 
可选，自定义评估函数（Evaluation Function），用于计算额外的评估指标。需返回一个 (name, value) 元组，其中 name 为指标名，value 为指标值。
#### 8.init_model: 
可选，已训练的 lightgbm.Booster 或保存的模型文件路径，用于继续训练或初始化模型。
#### 9.feature_name: 
可选，特征名称列表。默认为 'auto'，自动从 train_set 中获取特征名。
#### 10.categorical_feature:
 可选，类别特征索引列表或字典。默认为 'auto'，自动检测整数类型特征作为类别特征。
#### 11.early_stopping_rounds: 
可选，整数，早停轮数。当在指定轮数内验证集分数不再提升时停止训练。默认为 None，表示不启用早停。
#### 12.evals_result: 
可选，字典类型，用于存储每次迭代的评估结果。若提供，函数将填充此字典而不是输出到标准输出。
#### 13.verbose_eval: 
可选，布尔值或整数。默认为 True，表示在每个验证数据集上每 verbose_eval 轮输出一次评估结果。若为 False，则不输出。
#### 14.learning_rates: 
可选，列表或函数，用于指定每轮迭代的学习率。如果是一个列表，长度应等于 num_boost_round；如果是一个函数，将在每轮迭代时被调用以决定学习率。
#### 15.keep_training_booster: 
可选，布尔值。默认为 False，表示在训练结束后释放工作空间，节省内存。设为 True 则保留训练过程中使用的 Booster 对象，便于后续调用。
#### 16.callbacks: 
可选，回调函数列表，用于监控训练过程并在特定事件发生时执行操作。
#### 17.show_stdv: 
可选，布尔值。默认为 True，表示在输出评估结果时显示标准差（如果有多个数据并行验证）。

### lightgbm可视化
#### 1.特征重要性图
函数：lgb.plot_importance()
说明：绘制特征的重要性排名图，通常以特征的增益或覆盖度为度量标准。可以直观地看到哪些特征对模型预测最具影响力。
#### 2.学习曲线图

 - 函数：lgb.plot_metric()

说明：绘制训练过程中评估指标（如损失函数、AUC、准确率等）随迭代次数的变化情况。可用于观察模型是否过拟合、欠拟合，以及何时达到最优性能。
#### 3.特征分布直方图

 - 函数：lgb.plot_histogram()

说明：绘制单个特征在训练集、验证集（如有）和（或）测试集中的分布直方图，有助于对比不同数据集间特征分布的差异。
#### 4.特征分裂图

 - 函数：lgb.plot_split_value_histogram()

说明：针对指定特征，绘制其在决策树分裂时所采用的阈值分布直方图，揭示模型在训练过程中如何对该特征进行分割。
#### 5.决策树结构图

 - 函数：lgb.create_tree_digraph()（需安装 graphviz 库）

说明：生成单棵决策树的图形表示，展示树的层次结构、节点分裂条件和叶节点的预测值。通常需要配合 graphviz 库的 render 函数将生成的 DOT 文件转换为 PNG、SVG 等图像格式。
#### 6.特征分布与分裂点图

 - 函数：lgb.plot_tree_split_value_histogram()

说明：在同一图表中同时展示特征的分布直方图和决策树在该特征上的分裂点，直观呈现特征值与模型决策的关系。
#### 7.SHAP 贡献值分布图

 - 函数：shap.summary_plot()（需安装 shap 库）

说明：虽然不是 LightGBM 内置函数，但可以通过与 shap 库结合，绘制 SHAP（SHapley Additive exPlanations）值的分布图，展示每个特征对模型输出的平均效应及其分布情况。
这些函数通常返回 matplotlib 图形对象，可以直接显示或进一步定制。例如：

```python
import lightgbm as lgb
import matplotlib.pyplot as plt

# 假设已训练好一个 LightGBM 模型，并有相应的数据集
model = lgb.Booster(model_file='my_model.txt')
train_data = lgb.Dataset(X_train, y_train)

# 绘制特征重要性图
plt.figure(figsize=(10, 6))
lgb.plot_importance(model, max_num_features=20, importance_type='gain')
plt.title('Feature Importance (Gain)')
plt.show()

# 绘制学习曲线
evals_result = model.evals_result_
lgb.plot_metric(evals_result, metric='rmse')
plt.show()

plt.title('Learning Curve')
lgb.plot_histogram(train_data, feature='feature_name', bins=20)
plt.title('Feature Distribution')
plt.show()
```
