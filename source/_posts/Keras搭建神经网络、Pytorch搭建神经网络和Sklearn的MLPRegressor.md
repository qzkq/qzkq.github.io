---
title: Keras搭建神经网络、Pytorch搭建神经网络和Sklearn的MLPRegressor
date: 2026-01-01 08:11:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Keras搭建神经网络、Pytorch搭建神经网络和Sklearn的MLPRegressor)
# sklearn.neural_network.MLPRegressor 
scikit-learn 库中提供的一个多层感知器（Multi-Layer Perceptron, MLP）回归模型类，用于解决回归问题。该模型构建了一个包含隐藏层的前馈神经网络，并通过反向传播算法进行训练。以下是 MLPRegressor 主要参数的详细说明：
1. hidden_layer_sizes:
类型：元组或列表（整数）
默认值：(100,)
描述：定义神经网络的隐藏层结构。每个整数表示对应层中神经元的数量。例如，hidden_layer_sizes=(100, 50) 表示有两个隐藏层，第一个层有100个神经元，第二个层有50个神经元。如果只有一个整数，如 (100,)，则表示有一个单层、100个神经元的隐藏层。
2. activation:
类型：字符串
默认值：'relu'
描述：指定隐藏层和输出层神经元的激活函数。常见的选项包括：

> 'identity': 线性激活，即无激活（a(x) = x）。
>  'logistic': 逻辑sigmoid函数。 
>  'tanh': 双曲正切（hyperbolic tangent）函数。 
>  'relu': 常用的ReLU激活函数（Rectified Linear Unit），对于正输入返回原值，对于负输入返回0。 
>  'leaky_relu': Leaky ReLU，对负输入提供一个小的非零斜率。
> 'elu': Exponential Linear Unit (ELU) 激活函数。 
> 'selu': Scaled Exponential Linear Unit (SELU) 激活函数，适合自归一化神经网络。

3. solver:
类型：字符串
默认值：'adam'
描述：选择优化算法来调整权重。可选的值包括：

> 'lbfgs': 有限内存BFGS（L-BFGS）优化器。
>  'sgd': 随机梯度下降（Stochastic Gradient Descent）。 
> 'adam': Adam优化器，结合了动量和自适应学习率。
>  'rmsprop': RMSprop优化器。

4. alpha:
类型：浮点数
默认值：0.0001
描述：正则化强度（L2 regularization penalty）。较大的值有助于防止过拟合，但可能导致欠拟合。值为0表示不使用正则化。
5. batch_size:
类型：整数或 'auto'
默认值：'auto'
描述：当使用 mini-batch 优化算法（如 'sgd' 或 'adam'）时，指定每次迭代更新权重时使用的样本数量。若设置为 'auto'，则根据输入数据自动确定合适的批量大小。
6. learning_rate:
类型：字符串
默认值：'constant'
描述：指定学习率策略。不同的策略适用于不同的优化器。可选值包括：
'constant': 使用固定的 learning_rate_init。
'invscaling': 学习率随着迭代次数按比例递减。
'adaptive': 对于 'sgd' 和 'adam' 优化器，采用自适应学习率策略。
7. learning_rate_init:
类型：浮点数
默认值：0.001
描述：初始化的学习率。实际学习率会根据 learning_rate 参数指定的策略进行调整。
8. power_t:
类型：浮点数
默认值：0.5
描述：仅在使用 learning_rate='invscaling' 时生效，用于控制学习率衰减的速度。学习率与迭代次数的幂次成反比，幂指数由 power_t 决定。
9. max_iter:
类型：整数
默认值：200
描述：最大迭代次数。达到此次数后，训练过程将停止，即使没有达到收敛条件。
10. shuffle:
- 类型：布尔值
- 默认值：`True`
- 描述：在每次迭代开始时是否随机打乱训练数据。对于 mini-batch 更新，这有助于引入更多随机性并避免局部最优。
11. tol:
- 类型：浮点数
- 默认值：`1e-4`
- 描述：训练停止的容忍度。如果目标函数的改善小于 `tol`，则认为已收敛并停止训练。
12. verbose:
- 类型：整数或布尔值
- 默认值：`False`
- 描述：控制训练过程中信息的输出。非零整数表示输出训练进度的频率；`True` 等同于 `1`。
13. warm_start:
- 类型：布尔值
- 默认值：`False`
- 描述：是否在多次调用 `.fit()` 时重用上一次的模型参数作为初始值。启用后，可以在不丢失已有训练结果的情况下调整超参数或增加更多数据。
14. momentum:
- 类型：浮点数
- 默认值：`0.9`
- 描述：仅在使用 `'sgd'` 优化器时有效，表示动量因子，用于加速梯度下降过程。
15. nesterovs_momentum:
- 类型：布尔值
- 默认值：`True`
- 描述：仅在使用 `'sgd'` 优化器且 `momentum > 0` 时有效，决定是否使用 Nesterov 加速动量。
16. early_stopping:
- 类型：布尔值或字典
- 默认值：`False`
- 描述：启用早停机制以避免过拟合。如果为 `True`，将在验证集上监控损失并在性能不再改善时停止训练。也可以提供一个字典来指定早停参数，如 `{'patience': 10}`。
17. validation_fraction:
- 类型：浮点数（0, 1]
- 默认值：`0.1`
- 描述：仅在启用 `early_stopping` 时生效，表示用于验证集的训练数据比例。
18. beta_1 和 beta_2:
- 类型：浮点数
- 默认值：`0.9` 和 `0.999`
- 描述：仅在使用 `'adam'` 优化器时有效，分别是Adam算法中的第一阶和第二阶动量项的平滑系数。
19. epsilon:
- 类型：浮点数
- 默认值：`1e-8`
- 描述：仅在使用 `'adam'` 或 `'rmsprop'` 优化器时有效，防止数值计算时除以零。
20. random_state:
- 类型：整数、`RandomState` 实例、`None`
- 默认值：`None`
- 描述：控制随机数生成器的种子，确保实验的可重复性。
21. verbose:
- 类型：整数或布尔值
- 默认值：`False`
- 描述：控制训练过程中的输出信息级别。
22. warm_start:
- 类型：布尔值
- 默认值：`False`
- 描述：是否在多次调用 `.fit()` 时重用上一次的模型参数作为初始值。
23. momentum:
- 类型：浮点数
- 默认值：`0.9`
- 描述：仅在使用 `'sgd'` 优化器时有效，表示动量因子，用于加速梯度下降过程。
24. nesterovs_momentum:
- 类型：布尔值
- 默认值：`True`
- 描述：仅在使用 `'sgd'` 优化器且 `momentum > 0` 时有效，决定是否使用 Nesterov 加速动量。
25. early_stopping:
- 类型：布尔值或字典
- 默认值：`False`
- 描述：启用早停机制以避免过拟合。如果为 `True`，将在验证集上监控损失并在性能不再改善时停止训练。也可以提供一个字

# Keras搭建神经网络、Pytorch搭建神经网络和Sklearn的MLPRegressor
## 1.Sklearn的MLPRegressor
```python
clf = MLPRegressor(
	      solvers='adam',
	      activation='relu',
	      learning_rate='constant',
	      learning_rate_init=0.05,
	      shuffle=True,
	      random_state=6,
	      early_stopping=True,
	      validation_fraction=0.2
	  )
```
## 2.Keras搭建神经网络
```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
from tensorflow.keras.optimizers import Adam
from tensorflow.keras.callbacks import EarlyStopping

# 1. 定义模型结构
model = Sequential()
model.add(Dense(units=100, activation='relu', input_dim=input_shape))  # 假设 input_shape 是您的特征维度
model.add(Dense(units=1, activation='linear'))  # 输出层只有一个神经元，线性激活（默认）

# 2. 设置优化器和学习率
optimizer = Adam(learning_rate=0.05)

# 3. 编译模型
model.compile(loss='mean_squared_error', optimizer=optimizer, metrics=['mse'])

# 4. 设置早期停止回调
early_stopping = EarlyStopping(monitor='val_loss', patience=10, restore_best_weights=True)

# 5. 训练模型
history = model.fit(X_train, y_train, 
                    validation_split=0.2,  # 使用20%数据作为验证集
                    epochs=epochs,  # 您需要指定一个适当的epochs值，因为Keras不直接支持最大迭代次数
                    batch_size=batch_size,  # 如果需要，指定batch_size
                    shuffle=True,  # 在训练过程中打乱数据
                    callbacks=[early_stopping],  # 传入早期停止回调
                    verbose=1,  # 控制训练过程中的输出信息级别
                    random_state=6)  # 由于Keras内部使用tf.random.set_seed()，这里设置全局随机种子
```
注意：

 - Keras 不直接支持 max_iter 参数，因此您需要指定一个合适的 epochs（迭代次数）值。可以通过观察训练过程中的损失变化来确定一个合理的值。
 - Keras 的 EarlyStopping 回调没有直接对应 validation_fraction 参数，而是通过  validation_split 参数在 fit() 函数中指定验证集比例。这里的 patience=10 是一个示例值，表示在验证集损失连续10轮未改善时停止训练。您可以根据需求调整这个值。
 - Keras 使用 tf.random.set_seed() 设置全局随机种子以保证实验的可重复性。将 random_state=6 替换为 tf.random.set_seed(6) 即可实现相同效果。

### Dense参数
在Keras中，Dense层是一种全连接（Fully Connected, FC）层，它是神经网络中最基础且常用的层类型之一，负责将前一层的所有节点与当前层的所有节点之间建立全连接关系。
1. units
类型: int
作用: 指定该全连接层输出节点（或称神经元）的数量。这个数值直接影响了该层输出的维度。更多的节点通常意味着模型具有更强的表达能力和捕捉复杂模式的能力，但也可能导致过拟合和增加计算成本。
2. activation
类型: str 或 Activation 函数
默认值: None（无激活函数）
作用: 指定应用于该层输出的非线性激活函数。常见的激活函数有：

> 'relu'：整流线性单元（Rectified Linear Unit），常用作默认激活函数，用于处理非负值输入。
> 'sigmoid'：S型曲线函数，输出范围为(0, 1)，常用于二分类问题的最后一层。
> 'softmax'：归一化指数函数，输出为概率分布形式，常用于多分类问题的最后一层。 'tanh'：双曲正切函数，输出范围为(-1, 1)，适用于需要输出在双曲范围内的情况。

3. use_bias
类型: bool
默认值: True
作用: 控制是否为该层添加偏置项（bias）。偏置项是神经网络中每个节点独立的可训练参数，通常情况下启用偏置项能够增强模型的学习能力。
4. kernel_initializer
类型: str 或 Initializer 对象
作用: 指定该层权重矩阵（kernel或weights）的初始化方法。常见的初始化器如'glorot_uniform'（Xavier uniform初始化）、'glorot_normal'（Xavier normal初始化）、'lecun_uniform'、'lecun_normal'、'he_normal'（He initialization for ReLU-based activations）等，也可以使用自定义初始化器对象。
5. bias_initializer
类型: str 或 Initializer 对象
作用: 指定该层偏置项的初始化方法。常见的初始化器如'zeros'（所有偏置初始化为0）、'ones'、'uniform'、'normal'等，也可以使用自定义初始化器对象。
6. kernel_regularizer
类型: Regularizer 对象或 None
作用: 添加应用于权重矩阵的正则化项，用于防止过拟合。常见的正则化方法有L1、L2正则化，可以通过l1()、l2()等函数创建相应的正则化器对象。
7. bias_regularizer
类型: Regularizer 对象或 None
作用: 添加应用于偏置项的正则化项，与kernel_regularizer类似，用于防止过拟合。
8. activity_regularizer
类型: Regularizer 对象或 None
作用: 添加应用于该层输出的正则化项，基于层的输出值（激活值）计算正则化损失。
9. kernel_constraint
类型: Constraint 对象或 None
作用: 添加对权重矩阵的约束条件，如限制其最大值、最小值或范数等。
10. bias_constraint
类型: Constraint 对象或 None
作用: 添加对偏置项的约束条件，与kernel_constraint类似。
11. dtype
类型: str 或 tf.DType
作用: 指定该层数据类型的默认值。如果不指定，将继承自输入数据或全局默认值。
12. input_dim
类型: int
作用: （仅在第一层时使用）指定输入数据的特征维度，当模型没有明确的输入_shape时，可以使用此参数来指定。在后续层中，这一信息由前一层自动传递，无需显式指定。
综上所述，Keras中的Dense层通过这些参数提供了丰富的定制选项，使用户能够根据特定任务的需求调整模型结构、激活函数、初始化策略、正则化方法等，以优化模型性能和泛化能力。
## 3.Pytorch搭建神经网络
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
from torch.autograd import Variable
from torch.optim.lr_scheduler import StepLR
from sklearn.model_selection import train_test_split

# 1. 定义模型结构
class MLP(nn.Module):
    def __init__(self, input_dim, hidden_units=100):
        super(MLP, self).__init__()
        self.fc1 = nn.Linear(input_dim, hidden_units)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_units, 1)

    def forward(self, x):
        x = self.relu(self.fc1(x))
        return self.fc2(x)

model = MLP(input_dim=input_shape)  # 假设 input_shape 是您的特征维度

# 2. 设置优化器和学习率
optimizer = optim.Adam(model.parameters(), lr=0.05)

# 3. 准备数据
X_train_tensor = torch.tensor(X_train, dtype=torch.float32)
y_train_tensor = torch.tensor(y_train, dtype=torch.float32)

train_dataset = TensorDataset(X_train_tensor, y_train_tensor)
train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)

# 4. 定义训练循环
num_epochs = ...  # 您需要指定一个适当的epochs值，因为PyTorch不直接支持最大迭代次数
patience = 10  # 早停的耐心值

best_val_loss = float('inf')
num_no_improvement_epochs = 0

for epoch in range(num_epochs):
    for inputs, targets in train_loader:
        optimizer.zero_grad()
        outputs = model(inputs)
        loss = nn.MSELoss()(outputs, targets)
        loss.backward()
        optimizer.step()

    # 评估验证集损失（假设您已经有 X_val 和 y_val）
    val_inputs = torch.tensor(X_val, dtype=torch.float32)
    val_targets = torch.tensor(y_val, dtype=torch.float32)
    val_outputs = model(val_inputs)
    val_loss = nn.MSELoss()(val_outputs, val_targets)

    if val_loss < best_val_loss:
        best_val_loss = val_loss
        num_no_improvement_epochs = 0
    else:
        num_no_improvement_epochs += 1

    if num_no_improvement_epochs >= patience:
        print("Early stopping at epoch:", epoch)
        break

print("Training completed.")

```
注意以下几点：

 - PyTorch 不直接支持 max_iter 参数，因此您需要指定一个合适的 num_epochs（迭代次数）值。可以通过观察训练过程中的损失变化来确定一个合理的值。
 - PyTorch 没有内置的早期停止功能，您需要手动实现。这里展示了如何在训练循环中检查验证集损失，并在连续若干轮未改善时停止训练。patience 参数表示早停的耐心值，您可以根据需求调整这个值。
 - PyTorch 使用 torch.manual_seed() 设置全局随机种子以保证实验的可重复性。将 random_state=6 替换为 torch.manual_seed(6) 即可实现相同效果。
