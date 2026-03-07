---
title: Tabnet介绍（Decision Manifolds）和PyTorch TabNet之TabNetRegressor
date: 2026-01-03 08:11:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿
@[TOC](Tabnet介绍（Decision Manifolds）和PyTorch TabNet之TabNetRegressor)
# Decision Manifolds
指在决策树模型中，数据点通过一系列超平面的分割形成的决策边界。具体来说：

 - **在决策树模型中**：决策流形由一系列垂直于特征轴的超平面组成，这些超平面将数据空间划分为多个区域，每个区域代表一个决策区域。例如，一个简单的决策树可能通过比较特征值与某个阈值来决定数据点的分类或回归结果。
 -  **适用于表格数据**：由于表格数据通常具有结构化特征，决策流形的这种分割方式能够有效地捕捉数据中的线性关系，尤其是在特征维度较低的情况下，能够实现较好的分类或回归性能。
 - **可解释性**：决策流形的直观分割使得模型的决策过程易于理解，每个分割超平面都对应一个特定的特征阈值，便于人类解释和理解模型的决策依据。
 - **对比神经网络**：与依赖于高维非线性映射的神经网络不同，决策流形提供了一种更直接、更简单的决策方式，这在某些情况下使得决策树模型在表格数据上表现更佳。

此外，决策流形的概念也与模型的归纳偏差相关，即模型在学习过程中倾向于生成符合某种先验知识或规则的解。对于表格数据，决策树模型的决策流形天生具备线性分割的归纳偏差，这有助于它在没有过多参数调整的情况下，仍然能够有效地学习到数据的结构。

# TabNet
一种专门为结构化数据（表格数据）设计的深度学习模型，由 Google 提出。它通过注意力机制和可解释性设计，解决了传统神经网络在处理表格数据时透明度不足的问题。以下是 TabNet 的详细解析：
## 1.核心思想

 - **稀疏注意力机制**：TabNet 使用稀疏注意力机制来选择输入特征的子集进行处理，从而减少计算量并提高模型的可解释性。
 - **逐步特征选择**：模型逐步选择重要的特征，并忽略不相关的特征，这使得 TabNet 能够专注于对任务最重要的特征。
## 2. 架构组成
TabNet 的架构主要由以下几个部分组成：
 - Feature Transformer：负责对输入特征进行非线性变换。
 - Attention Mechanism：通过注意力机制选择重要的特征子集。
 - Masking Mechanism：生成掩码，决定哪些特征被选中参与下一步计算。
 - Decoder：用于预测任务（如分类或回归）。
## 3. 工作流程
 - 输入层：将表格数据输入到模型中。
 - 特征变换：通过 Feature Transformer 对特征进行非线性变换。
 - 注意力选择：使用注意力机制选择重要的特征子集。
 - 掩码生成：生成掩码以决定哪些特征参与下一步计算。
 - 输出层：通过 Decoder 输出预测结果。
## 4. 优点
 - 可解释性：由于稀疏注意力机制，TabNet 可以明确指出哪些特征对预测结果有贡献。
 - 高效性：通过选择重要特征，减少了不必要的计算。
 - 灵活性：适用于多种任务，包括分类和回归。

# PyTorch TabNet
pytorch_tabnet 是基于 PyTorch 实现的 TabNet 模型库，专为结构化数据（表格数据）设计。它提供了高效的特征选择和可解释性功能，适用于分类和回归任务。

## TabNetRegressor参数

`TabNetRegressor` 是一种基于 TabNet 架构的回归模型，适用于结构化数据的回归任务。以下是其主要参数的详细说明：

---

### 1. 模型相关参数
#### `n_d`
- **类型**: int  
- **默认值**: 8  
- **描述**: 表示决策路径中每个步骤的维度大小。较大的值会增加模型的表达能力，但也可能导致过拟合。

#### `n_a`
- **类型**: int  
- **默认值**: 8  
- **描述**: 表示注意力机制的维度大小。与 `n_d` 类似，控制模型的复杂度。

#### `n_steps`
- **类型**: int  
- **默认值**: 3  
- **描述**: 表示 TabNet 模型中的步数（steps），即模型在每轮迭代中选择特征的次数。更大的值可以捕获更多的特征组合。

####  `gamma`
- **类型**: float  
- **默认值**: 1.3  
- **描述**: 控制特征稀疏性的超参数。较大的值会导致更少的特征被选中。

####  `cat_idxs`
- **类型**: list[int]  
- **默认值**: []  
- **描述**: 指定分类特征的索引列表。如果数据集中包含分类变量，需要通过此参数指定。

####  `cat_dims`
- **类型**: list[int]  
- **默认值**: []  
- **描述**: 指定分类特征的类别数量。与 `cat_idxs` 配合使用，用于定义分类变量的嵌入维度。

####  `cat_emb_dim`
- **类型**: int 或 list[int]  
- **默认值**: 1  
- **描述**: 分类特征的嵌入维度。如果为整数，则所有分类特征共享相同的嵌入维度；如果为列表，则每个分类特征可以有不同的嵌入维度。

---

### 2. 训练相关参数
#### `optimizer_fn`
- **类型**: function  
- **默认值**: Adam  
- **描述**: 优化器函数，默认使用 PyTorch 的 Adam 优化器。

#### `optimizer_params`
- **类型**: dict  
- **默认值**: {'lr': 0.02}  
- **描述**: 传递给优化器的参数字典，例如学习率（`lr`）。

#### `scheduler_fn`
- **类型**: function  
- **默认值**: None  
- **描述**: 学习率调度器函数。如果需要动态调整学习率，可以通过此参数指定。

#### `scheduler_params`
- **类型**: dict  
- **默认值**: None  
- **描述**: 传递给学习率调度器的参数字典。

#### `mask_type`
- **类型**: str  
- **默认值**: "sparsemax"  
- **描述**: 特征选择掩码的类型，可选值为 `"sparsemax"` 和 `"entmax"`。`"sparsemax"` 更加常用。

mask_type 参数用于指定特征选择掩码的类型，控制模型在每个决策步骤中选择特征的方式。

---

### 3. 其他参数
#### `seed`
- **类型**: int  
- **默认值**: 0  
- **描述**: 随机种子，用于确保结果的可重复性。

#### `verbose`
- **类型**: int  
- **默认值**: 1  
- **描述**: 控制输出日志的详细程度。`0` 表示静默模式，`1` 表示普通模式，`2` 表示调试模式。

#### `device_name`
- **类型**: str  
- **默认值**: "auto"  
- **描述**: 指定计算设备。`"auto"` 会自动检测是否有 GPU 可用。

---

## TabNetRegressor.fit 参数详解
### 1. 核心训练数据参数

#### `X_train`

 - **必须为 numpy.ndarray 格式，不支持直接传入 pandas.DataFrame 或 pandas.Series。**

#### `y_train`
- **必须为 numpy.ndarray 格式，且形状需调整为 (n_samples, 1)。**
---

### 2. 验证数据参数
#### `eval_set`
- **类型**: list[tuple]  
- **默认值**: None  
- **描述**: 验证集数据列表，格式为 `[(X_valid, y_valid)]`。支持多个验证集。

#### `eval_name`
- **类型**: list[str]  
- **默认值**: None  
- **描述**: 每个验证集的名称，便于在日志中区分不同验证集。

#### `eval_metric`
- **类型**: list[str] 或 callable  
- **默认值**: ['rmse']  
- **描述**: 评估指标，可选值包括 `'rmse'`、`'mse'` 等。也可以传入自定义的评估函数。

---

### 3. 训练控制参数
#### `max_epochs`
- **类型**: int  
- **默认值**: 100  
- **描述**: 最大训练轮数。如果提前收敛，则可能在达到最大轮数之前停止。

#### `patience`
- **类型**: int  
- **默认值**: 10  
- **描述**: 早停机制的耐心值。如果验证集性能在连续 `patience` 轮内没有提升，则停止训练。

#### `batch_size`
- **类型**: int  
- **默认值**: 1024  
- **描述**: 每次迭代的批量大小。较大的批量可能会加速训练，但需要更多的内存。

#### `virtual_batch_size`
- **类型**: int  
- **默认值**: 128  
- **描述**: 虚拟批量大小，用于模拟小批量梯度下降，减少内存占用。

#### `num_workers`
- **类型**: int  
- **默认值**: 0  
- **描述**: 数据加载器中的工作线程数。设置为 0 表示使用主进程加载数据。

---

### 4. 回调与日志参数
#### `drop_last`
- **类型**: bool  
- **默认值**: False  
- **描述**: 是否丢弃最后一个不完整的批量数据。

#### `callbacks`
- **类型**: list[Callback]  
- **默认值**: None  
- **描述**: 自定义回调函数列表，例如学习率调度器、早停等。

#### `from_unsupervised`
- **类型**: TabNetPretrainer  
- **默认值**: None  
- **描述**: 如果提供了预训练的 `TabNetPretrainer` 模型，则从无监督预训练阶段继续训练。

---
#### `loss_fn ` 
- **类型**: Callable（可调用对象，例如函数或类方法）
- **默认值**: 默认使用均方误差（MSE）损失函数。
- **描述**: 允许用户自定义训练过程中使用的损失函数。
---

## TabNetRegressor.predict 参数
### 1. 核心参数
#### `X`
- **数据格式应与训练时使用的 `X_train` 一致。**

#### `batch_size`
- **类型**: int  
- **默认值**: 1024  
- **描述**: 每次预测的批量大小。较大的批量可能会加速预测过程，但需要更多的内存。

#### `num_workers`
- **类型**: int  
- **默认值**: 0  
- **描述**: 数据加载器中的工作线程数。设置为 0 表示使用主进程加载数据。

#### `from_unsupervised`
- **类型**: TabNetPretrainer  
- **默认值**: None  
- **描述**: 如果提供了预训练的 `TabNetPretrainer` 模型，则从无监督预训练阶段继续预测。

#### `return_proba`
- **类型**: bool  
- **默认值**: False  
- **描述**: 是否返回预测的概率分布（仅适用于分类任务）。对于回归任务，此参数无效。

#### `verbose`
- **类型**: int  
- **默认值**: 1  
- **描述**: 控制输出日志的详细程度。`0` 表示静默模式，`1` 表示普通模式，`2` 表示调试模式。

---

### 2. 返回值

- **当 TabNetRegressor.predict 返回的预测结果是一个二维数组（例如 (n_samples, 1)）时，可以使用 flatten 方法将其转换为一维数组 (n_samples,)。**

---

```python
skf = StratifiedKFold(n_splits=n_splits, shuffle=True, random_state=2025)
for trn_idx, val_idx in skf.split(train_feats[feature_names], train_feats['V_bins']):
    X_train = train_feats.loc[trn_idx][feature_names].values
    Y_train = train_feats.loc[trn_idx]['V'].values.reshape(-1, 1)
    X_val = train_feats.loc[val_idx][feature_names].values
    Y_val = train_feats.loc[val_idx]['V'].values.reshape(-1, 1)
    print("Train Num: ", len(Y_train))
    print("Val Num: ", len(Y_val))
    model = tab_model.TabNetRegressor(
                                  n_d = 8,
                                  n_a = 8,
                                  n_steps = 1,
                                  gamma = 1.6,
                                  lambda_sparse = 6e-5,
                                  n_independent = 4,
                                  n_shared = 2,
                                  optimizer_fn = torch.optim.AdamW,
                                  optimizer_params = dict(lr=0.025),
                                  scheduler_fn = torch.optim.lr_scheduler.ReduceLROnPlateau,
                                  scheduler_params = dict(mode='min', factor=0.6, patience=3),
                                  mask_type = 'entmax',
                                  seed=2025,
                                  device_name = 'cuda',
                                  verbose = 1,
                                 )
    model.fit(
        X_train=X_train,
        y_train=Y_train,
        eval_set=[(X_val, Y_val)],
        eval_name=['val'],
        eval_metric=['rmse'],
        patience=10,
        max_epochs=100,
        batch_size=512, 
        virtual_batch_size=128, 
        num_workers=1, 
        drop_last=False,
    )
    pred = model.predict()
    pred = pred.flatten()
```

# 参考 
1.[https://github.com/dreamquark-ai/tabnet](https://github.com/dreamquark-ai/tabnet)
2.[https://github.com/google-research/google-research/tree/master/tabnet](https://github.com/google-research/google-research/tree/master/tabnet)
3.[https://arxiv.org/abs/1908.07442](https://arxiv.org/abs/1908.07442)
