---
title: Pytorch详解-模型模块(RNN,CNN,FNN,LSTM,GRU,TCN,Transformer)
date: 2026-01-03 08:07:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Pytorch详解-模型模块)

## Module & parameter
**定义模型类**

 - 继承 nn.Module： 模型类通常继承自 nn.Module 类。
 - 初始化方法 __init__： 在这个方法中，定义模型的层（例如线性层、卷积层等）。
 - 前向传播方法 forward： 定义数据通过模型的流动方式
### Module初认识
在pytorch中模型是一个Module，各网络层、模块也是Module。Module是所有神经网络的基类，所有的模型都必须继承于Module类，并且它可以嵌套，一个Module里可以包含另外一个Module。

在 PyTorch 中，nn.Module 类使用多个有序字典来管理其内部状态和功能。这些有序字典主要用于跟踪模型中的各种组件，如子模块、参数、缓冲区等。

 - _modules:

类型: OrderedDict
用途: 存储模型的子模块（即其他 nn.Module 实例）。每个子模块都有一个唯一的名称作为键。

 - _parameters:

类型: OrderedDict
用途: 存储模型的所有可学习参数。这些参数通常是 torch.Tensor 对象，并且需要梯度计算。

 - _buffers:

类型: OrderedDict
用途: 存储模型的缓冲区，这些缓冲区包含不需要梯度计算的数据，比如 BatchNorm 层中的运行平均值和方差。

 - _non_persistent_buffers_set:

类型: set
用途: 存储不需要持久化的缓冲区的名称集合。这些缓冲区不会被保存到模型的状态字典中。

 - _backward_hooks:

类型: OrderedDict
用途: 存储反向传播时的钩子函数。这些钩子可以修改梯度或执行其他操作。

 - _state_dict_hooks:

类型: OrderedDict
用途: 存储在序列化模型状态字典时调用的钩子函数。这些钩子允许用户自定义如何保存和加载模型的状态。

 - _load_state_dict_pre_hooks:

类型: OrderedDict
用途: 存储在从状态字典加载模型之前调用的钩子函数。这些钩子可以修改状态字典的内容。

 - _load_state_dict_post_hooks:

类型: OrderedDict
用途: 存储在从状态字典加载模型之后调用的钩子函数。这些钩子可以在模型加载完成后执行一些操作。
#### forward函数
forward之于Module等价于getitem之于Dataset。forward函数是模型每次调用的具体实现，所有的模型必须实现forward函数，否则调用时会报错。

 - 自动调用：

当你创建了一个 nn.Module 的实例，并将其当作函数调用时（例如 model(input)），实际上是在调用该实例的 forward 方法。这是因为 nn.Module 类中定义了一个特殊的 \__call__ 方法，它会自动调用 forward 方法。

**Module是所有模型的基类**

 - 每个module有8个字典管理它的核心属性
 - 一个module可以包含多个子module
 - 一个module相当于一个运算，必须实现forward函数

### Parameter

在 PyTorch 中，Parameter 是一个特殊类型的 torch.Tensor，用于表示模型中的可学习参数。这些参数通常是模型中的权重和偏置项，它们会在训练过程中通过反向传播算法进行更新。

 - Parameter 类实际上是 torch.Tensor 的子类。
 - 当你在 nn.Module 的子类中定义一个 Parameter 时，它会被自动添加到模型的 _parameters 字典中

#### Pytorch中的权重、参数和超参数
**参数（Parameters）**

 - 定义：
	 - 参数是模型的一部分，它们是在训练过程中通过优化算法学习得到的。
	 - 在 PyTorch 中，参数通常通过 nn.Module 的子类定义，并且是 Parameter 类型的对象。 参数存储在模型的 _parameters 字典中。
 - 用途：
	 - 参数用于定义模型的输出，即模型如何从输入映射到输出。
	 - 在训练过程中，参数通过反向传播算法根据损失函数的梯度进行更新。
 - 示例：
	 - 在神经网络中，权重矩阵和偏置向量是参数的例子。
	 - 在 PyTorch 中，这些参数通常通过 nn.Linear、nn.Conv2d 等层定义。

**权重（Weights）**

 - 定义：
	 - 权重是参数的一种，特指连接神经网络中各层节点的数值。
	 - 在 PyTorch 中，权重通常是指 nn.Module 中定义的 Parameter 类型的对象，尤其是那些代表连接权重的参数。
 - 用途：
	 - 权重用于控制输入特征对输出的影响程度。
	 - 在训练过程中，权重通过反向传播算法进行更新以最小化损失函数。
 - 示例：
	 - 在多层感知机（MLP）中，每一层之间的连接都有对应的权重。
	 - 在卷积神经网络（CNN）中，卷积核的系数也是权重。

**超参数（Hyperparameters）**

 - 定义：
	 - 超参数是在模型训练开始之前设置的参数，它们不是通过训练过程学习得到的。
	 - 在 PyTorch 中，超参数通常需要手动设置，并且用于控制模型的训练过程，包括训练的速度、复杂度和稳定性。
 - 用途：
	 - 超参数用于指导模型的学习过程，例如学习率、批次大小、正则化系数等。
	 - 通常需要通过实验来调整超参数以获得更好的模型性能。
 - 示例：
	 - 学习率（Learning Rate）：控制权重更新的步长。
	 - 批次大小（Batch Size）：每次更新权重时使用的样本数量。
	 - 正则化参数（Regularization Parameter）：用于控制模型复杂度，防止过拟合。

在 PyTorch 中，nn.Module 类使用 _parameters 这个有序字典来管理模型中的所有可学习参数。这些参数通常是模型中的权重和偏置项，它们会在训练过程中通过反向传播算法进行更新。

### Module容器-Containers

在深度学习模型里面，有一些网络层需要放在一起使用，如 conv + bn + relu 的组合。Module的容器是将一组操作捆绑在一起的工具，在pytorch官方文档中把Module也定义为Containers，或许是因为“Modules can also contain other Modules”。

#### Sequential

**定义：**

 - nn.Sequential 是一个特殊的 nn.Module 类，用于按顺序组织多个 nn.Module 实例。
 - 它可以接受任意数量的子模块作为参数，并按照它们被传入的顺序执行。

**用途：**

 - 用于构建简单的前馈网络，其中层之间按照定义的顺序依次传递数据。
 - 适用于不需要复杂分支或循环结构的情况。

**构造函数：**

 - 构造函数可以接受任意数量的 nn.Module 实例作为参数。

**前向传播：**

 - nn.Sequential 的 forward 方法会依次调用每个子模块的 forward 方法。
 - 输入数据会从第一个子模块开始传递，直到最后一个子模块。

#### ModuleList
**定义：**

 - nn.ModuleList 是一个可变长度的列表，可以包含任意数量的子模块。
 - 它可以用于构建动态结构的模型，例如循环神经网络（RNN）中的多个时间步。

**用途：**

 - 用于构建动态结构的模型，例如循环神经网络（RNN）中的多个时间步。
 - 适用于需要灵活控制子模块的场景，例如根据不同的条件选择不同的层。

**构造函数：**

 - 构造函数可以接受一个列表作为参数，其中包含 nn.Module 实例。

**索引和迭代：**

 - 可以像操作 Python 列表一样索引和迭代 nn.ModuleList。
 - 例如，可以通过索引访问单个子模块，或者使用 for 循环遍历所有子模块。

#### ModuleDict
**定义：**

 - nn.ModuleDict 是一个字典，键为字符串，值为 nn.Module 实例。
 - 它可以用于构建具有条件分支或可选择层的模型。

**用途：**

 - 用于构建具有条件分支或可选择层的模型。
 - 适用于需要根据不同的条件选择不同层的场景。

**构造函数：**

 - 构造函数可以接受一个字典作为参数，其中键为字符串，值为 nn.Module 实例。

**索引和迭代：**

 - 你可以像操作 Python 字典一样索引和迭代 nn.ModuleDict。
 - 例如，可以通过键访问单个子模块，或者使用 for 循环遍历所有子模块。

#### ParameterList & ParameterDict
nn.ParameterList 和 nn.ParameterDict 是 PyTorch 中用于管理一组参数的容器类。它们类似于 nn.ModuleList 和 nn.ModuleDict，但是专门用于存储和管理 nn.Parameter 实例。

**nn.ParameterList**
**定义**：

 - nn.ParameterList 是一个可变长度的列表，可以包含任意数量的 nn.Parameter 实例。

**用途：**

 - 用于管理一组可变长度的参数，例如在构建模型时需要显式地管理一组参数。

**构造函数：**

 - 构造函数可以接受一个列表作为参数，其中包含 nn.Parameter 实例。

**nn.ParameterDict**
**定义：**

 - nn.ParameterDict 是一个字典，键为字符串，值为 nn.Parameter 实例。

**用途：**

 - 用于管理一组以字符串为键的参数，例如在构建模型时需要根据条件选择不同的参数。

**构造函数：**

 - 构造函数可以接受一个字典作为参数，其中键为字符串，值为 nn.Parameter 实例。


## 常用网络层
### LSTM
torch.nn.LSTM 是 PyTorch 中用于创建 LSTM（长短时记忆）网络的一个模块。

```python
nn.LSTM(input_size, hidden_size, num_layers=1, bias=True, batch_first=False, dropout=0, bidirectional=False)
```

 - input_size: 输入数据的特征数。例如，如果你的输入数据是由词嵌入组成的，那么 input_size 就是词嵌入的维度。
 - hidden_size: LSTM 单元中隐藏状态的特征数。这同时也是 LSTM 层输出的特征数。 
 - num_layers: LSTM 层的数目。默认值为 1。如果设置为大于 1，则 LSTM 层会堆叠起来，形成一个多层 LSTM 结构。
 -  bias: 如果设置为 True，则 LSTM 单元会使用偏置项。默认值为 True。
 - batch_first: 如果设置为 True，则输入和输出数据的第一维度将是批大小。这意味着输入数据的形状应该是 (batch, sequence, features) 而不是 (sequence, batch, features)。默认值为 False。 
 - dropout: 当 num_layers > 1 时，应用于所有 LSTM 层之间的输出（除了最后一个 LSTM 层）的 dropout 概率。默认值为 0，意味着没有 dropout。
 -  bidirectional: 如果设置为 True，则 LSTM 层会变成双向的，即它会在正向和反向上同时处理序列。默认值为 False。

#### 输入和输出
**输入**

```python
input: 输入张量。如果 batch_first 为 True，则形状应为 (batch, seq_len, input_size)；否则形状应为 (seq_len, batch, input_size)。
(h_0, c_0): 初始隐藏状态和初始单元状态。形状为 (num_layers * num_directions, batch, hidden_size)。如果未提供，则使用零张量初始化。
```
bidirectional 参数：

 - 如果设置为 False（默认值），则 LSTM 为单向 LSTM，此时 num_directions 为 1。
 - 如果设置为 True，则 LSTM 为双向 LSTM，此时 num_directions 为 2。

**输出**

```python
output: LSTM 层的输出。形状与输入相同，但最后一个维度是 hidden_size（对于单向 LSTM）或 hidden_size * 2（对于双向 LSTM）。
(h_n, c_n): 最终的隐藏状态和单元状态。形状为 (num_layers * num_directions, batch, hidden_size)。
```

```python
import torch.nn as nn

class PowerForecastDataset(Dataset):
    def __init__(self, data, seq_len=3):
        self.data = data
        self.seq_len = seq_len
        self.X, self.y = [], []
        
        # 归一化数据
        self.mean = np.mean(data, axis=0)
        self.std = np.std(data, axis=0)
        normalized_data = (data - self.mean) / self.std
        
        for i in range(len(normalized_data) - seq_len):
            self.X.append(normalized_data[i:(i+seq_len)])
            self.y.append(normalized_data[i+seq_len, -1])  # 假设最后一列是目标变量

    def __len__(self):
        return len(self.X)

    def __getitem__(self, idx):
        return torch.tensor(self.X[idx], dtype=torch.float), torch.tensor(self.y[idx], dtype=torch.float)


class LSTMModel(nn.Module):
    def __init__(self, input_dim, hidden_dim, layer_dim, output_dim):
        super(LSTMModel, self).__init__()
        self.hidden_dim = hidden_dim
        self.layer_dim = layer_dim
        self.lstm = nn.LSTM(input_dim, hidden_dim, layer_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        h0 = torch.zeros(self.layer_dim, x.size(0), self.hidden_dim).requires_grad_()
        c0 = torch.zeros(self.layer_dim, x.size(0), self.hidden_dim).requires_grad_()
        out, (hn, cn) = self.lstm(x, (h0.detach(), c0.detach()))
        out = self.fc(out[:, -1, :])
        return out
```
### GRU
torch.nn.GRU 是 PyTorch 中实现门控循环单元（Gated Recurrent Unit, GRU）的一个模块。GRU 是一种简化版的 LSTM（长短期记忆网络），旨在减少计算成本的同时保持对长期依赖的有效建模能力。
**参数说明**

 - input_size: 输入张量中的特征维度大小。这是每个时间步的输入向量的维度。
 - hidden_size: 隐层张量中的特征维度大小。这是 GRU 单元内部状态的维度，也是输出的维度。
 - num_layers: GRU 层的数量。可以堆叠多个 GRU 层以形成更深的网络结构。
 - bias: 如果为 True，则在门和候选隐藏状态中使用偏置项。默认为 True。
batch_first: 如果为 True，则输入和输出数据的第一个维度是批大小；否则，第一个维度是序列长度。默认为 False。
dropout: 当 num_layers > 1 时，在每两个 GRU 层之间使用的 dropout 概率。默认为 0（不使用 dropout）。
bidirectional: 如果为 True，则使用双向 GRU，即正向和反向两个方向上的 GRU 层。默认为 False。

**输入**
 - input: 形状为 (seq_len, batch, input_size) 或 (batch, seq_len, input_size) 的张量，取决于 batch_first 参数。
 - h_0: 形状为 (num_layers * num_directions, batch, hidden_size) 的初始隐藏状态。
   
  **输出**
 - output: 形状为 (seq_len, batch, hidden_size * num_directions) 或 (batch,  seq_len, hidden_size * num_directions) 的张量，取决于 batch_first 参数。
 - h_n: 形状为 (num_layers * num_directions, batch, hidden_size) 的最终隐藏状态。

```python
class GRUNet(nn.Module):
    def __init__(self, input_dim, hidden_dim, layer_dim, output_dim):
        super(GRUNet, self).__init__()
        self.hidden_dim = hidden_dim
        self.layer_dim = layer_dim
        self.gru = nn.GRU(input_dim, hidden_dim, layer_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        h0 = torch.zeros(self.layer_dim, x.size(0), self.hidden_dim).requires_grad_()
        out, hn = self.gru(x, h0.detach())
        out = self.fc(out[:, -1, :])
        return out
```

### Convolutional Layers
卷积层通过在输入数据上应用一系列的小型滤波器（也称为卷积核）来检测局部特征。
#### 卷积层的基本概念
**卷积核（Kernel / Filter）：**

 - 卷积核是一个小的矩阵，通常比输入数据的维度小得多。
 - 卷积核在输入数据上滑动，并与局部区域进行元素乘法和求和运算，得到新的特征图。

**步长（Stride）：**

 - 步长决定了卷积核在输入数据上移动的距离。
 - 较大的步长可以增加感受野，但会减少输出特征图的大小。

**填充（Padding）：**

 - 在输入数据周围添加零填充可以控制输出特征图的大小。
 - 常用的填充类型有“same”（保持输出特征图与输入相同大小）和“valid”（不添加任何填充）。

**通道数（Channels）：**

 - 输入数据通常有多个通道（例如 RGB 图像有三个通道）。
 - 卷积核的数量决定了输出特征图的数量。

**组卷积（Grouped Convolution）：**

 - 在某些情况下，可以将输入通道分成多个组，并为每个组独立应用卷积核。

 - 这种方法可以减少计算量，同时保持一定的表达能力。

#### 常见的卷积层类型
**二维卷积层（2D Convolutional Layer）：**

 - 最常用的类型，用于处理图像数据。
 - 在 PyTorch 中，可以使用 nn.Conv2d 来实现。

**一维卷积层（1D Convolutional Layer）：**

 - 用于处理序列数据，如时间序列信号。

 - 在 PyTorch 中，可以使用 nn.Conv1d 来实现。

**三维卷积层（3D Convolutional Layer）：**

 - 用于处理视频数据或三维体积数据。

 - 在 PyTorch 中，可以使用 nn.Conv3d 来实现。

**转置卷积层（Transposed Convolutional Layer）：**

 - 也称为分数跨步卷积（Fractionally-strided convolution）或反卷积（Deconvolution）。

 - 用于上采样，通常用于生成模型或语义分割任务。

 - 在 PyTorch 中，可以使用 nn.ConvTranspose2d 来实现。

**深度可分离卷积（Depthwise Separable Convolution）：**

 - 分为两步：首先对每个输入通道单独应用卷积，然后对结果进行逐点卷积。

 - 通常用于移动设备上的高效模型，如 MobileNet。

 - 在 PyTorch 中，可以组合使用 nn.Conv2d 和 nn.Conv2d 来实现。

```python
torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True, padding_mode='zeros')
```

torch.nn.Conv2d 是 PyTorch 中用于实现二维卷积层的模块。
**参数说明**

 - in_channels：
	 - 输入数据的通道数。
	 - 例如，对于 RGB 图像，in_channels 为 3；对于灰度图像，in_channels 为 1。
 - out_channels：
	 - 输出数据的通道数，即卷积核的数量。
	 - 每个卷积核会产生一个输出通道。
 - kernel_size：
	 - 卷积核的大小，可以是整数或元组。
	 - 如果是整数，则表示正方形卷积核的边长；如果是元组，则表示 (高度, 宽度)。
 - stride：
	 - 卷积核在输入数据上移动的步长，默认为 1。
	 - 可以是整数或元组 (步长高度, 步长宽度)。
 - padding：
	 - 输入数据边缘的填充大小，默认为 0。
	 - 可以是整数或元组 (填充高度, 填充宽度)。
	 - 如果设置为 "same"，则会自动计算所需的填充大小以保持输出特征图的大小与输入相同。
 - dilation：
	 - 卷积核中元素之间的间距，默认为 1。
	 - 较大的 dilation 可以扩大感受野，但会减少参数数量。
 - groups：
	 - 输入通道和输出通道的分组数，默认为 1。
	 - 当 groups > 1 时，表示进行组卷积，每个组内的输入通道和输出通道相互独立。
 - bias：
	 - 是否为每个输出通道添加偏置项，默认为 True。
	 - 如果为 False，则不会添加偏置项。
 - padding_mode：
	 - 填充模式，默认为 "zeros"。
	 - 可选的填充模式还包括 "reflect" 和 "replicate"。
![nn.Conv2d图像领域计算公式](/img/0a5b43561db1410b9ddbf95190973cac.png)
![在这里插入图片描述](/img/f8de443416044a4fb6b964d9f0b1208a.png)

### Pooling Layers
池化层（Pooling Layers）是卷积神经网络（Convolutional Neural Networks, CNNs）中的一个重要组成部分，主要用于减少特征图的尺寸，从而降低计算复杂度并帮助模型学习更具鲁棒性的特征表示。
#### 池化层的基本概念
**池化窗口（Pooling Window）：**

 - 池化操作在一个固定大小的窗口内进行。
 - 窗口大小通常小于输入特征图的尺寸。

**步长（Stride）：**

 - 池化窗口在输入特征图上移动的步长。
 - 较大的步长可以进一步减小输出特征图的尺寸。

**填充（Padding）：**

 - 在输入特征图周围添加零填充可以控制输出特征图的尺寸。
 - 通常情况下，池化层不使用填充。

**池化类型：**

 - 最大池化（Max Pooling）：取窗口内的最大值作为输出。
 - 平均池化（Average Pooling）：取窗口内的平均值作为输出。

#### 常见的池化层类型

**二维池化层（2D Pooling Layer）：**

 - 最常用的类型，用于处理图像数据。
 - 在 PyTorch 中，可以使用 nn.MaxPool2d 和 nn.AvgPool2d 来实现。

**一维池化层（1D Pooling Layer）：**

 - 用于处理序列数据，如时间序列信号。

 - 在 PyTorch 中，可以使用 nn.MaxPool1d 和 nn.AvgPool1d 来实现。

**三维池化层（3D Pooling Layer）：**

 - 用于处理视频数据或三维体积数据。

 - 在 PyTorch 中，可以使用 nn.MaxPool3d 和 nn.AvgPool3d 来实现。

```python
torch.nn.MaxPool2d(kernel_size, stride=None, padding=0, dilation=1, return_indices=False, ceil_mode=False)
```

torch.nn.MaxPool2d 是 PyTorch 中用于实现二维最大池化（Max Pooling）操作的模块。最大池化是一种常用的下采样技术，用于减少特征图的尺寸，从而降低计算复杂度并帮助模型学习更具鲁棒性的特征表示。

**参数说明**

 - kernel_size：
	 - 池化窗口的大小，可以是整数或元组。
	 - 如果是整数，则表示正方形池化窗口的边长；如果是元组，则表示 (高度, 宽度)。
 - stride：
	 - 池化窗口在输入数据上移动的步长，默认为 None。
	 - 如果 stride 为 None，则默认等于 kernel_size。
	 - 可以是整数或元组 (步长高度, 步长宽度)。
 - padding：
	 - 输入数据边缘的填充大小，默认为 0。
	 - 可以是整数或元组 (填充高度, 填充宽度)。
 - dilation：
	 - 池化窗口中元素之间的间距，默认为 1。
	 - 较大的 dilation 可以扩大感受野，但会减少参数数量。
 - return_indices：
	 - 是否返回最大值的索引，默认为 False。
	 - 如果为 True，则会返回最大值的位置索引，这在反向池化（unpooling）中有用。

![池化层输出特征图的大小计算公式](/img/3343c1cb877c4613a56126179d412563.png)
### 自适应池化层
自适应池化层（Adaptive Pooling Layers）是 PyTorch 中的一种特殊类型的池化层，它可以自动调整池化窗口的大小以确保输出特征图具有指定的大小。这种类型的池化层特别适用于需要固定输出尺寸的情况，比如在构建卷积神经网络时，当输入图像的尺寸变化时，自适应池化层可以保证网络输出的一致性。
#### 自适应池化层的基本概念
**输出尺寸（Output Size）：**

 - 指定输出特征图的尺寸。
 - 对于二维自适应池化层，输出尺寸通常是一个整数或元组 (height, width)。

**池化类型：**

 - 最大池化（Adaptive Max Pooling）：取窗口内的最大值作为输出。
 - 平均池化（Adaptive Average Pooling）：取窗口内的平均值作为输出。

#### 常见的自适应池化层类型

**二维自适应池化层（2D Adaptive Pooling Layer）：**

 - 最常用的类型，用于处理图像数据。
 - 在 PyTorch 中，可以使用 nn.AdaptiveMaxPool2d 和 nn.AdaptiveAvgPool2d 来实现。

**一维自适应池化层（1D Adaptive Pooling Layer）：**

 - 用于处理序列数据，如时间序列信号。

 - 在 PyTorch 中，可以使用 nn.AdaptiveMaxPool1d 和 nn.AdaptiveAvgPool1d 来实现。

**三维自适应池化层（3D Adaptive Pooling Layer）：**

 - 用于处理视频数据或三维体积数据。

 - 在 PyTorch 中，可以使用 nn.AdaptiveMaxPool3d 和 nn.AdaptiveAvgPool3d 来实现。


### Padding Layers
填充层（Padding Layers）在深度学习中用于在输入数据周围添加额外的数据，通常是零值，以改变输入数据的尺寸。填充层有助于保持卷积操作后输出特征图的尺寸不变，或者用于扩大输入数据的边界，使得边界上的像素也能被卷积核充分考虑。
#### 基本概念
**填充（Padding）：**

 - 在输入数据周围添加的数据。
 - 通常情况下，填充使用的是零值，但也支持其他类型的填充。

**填充大小（Padding Size）：**

 - 指定每个维度上填充的大小。

 - 可以是整数或元组 (left, right, top, bottom)。

**填充模式（Padding Mode）：**

 - 填充使用的模式。

 - 常见的模式包括 "zeros"、"reflect" 和 "replicate"。

#### 常见的填充层类型
**二维填充层（2D Padding Layer）：**

 - 用于处理图像数据。
 - 在 PyTorch 中，可以使用 nn.ZeroPad2d、nn.ReflectionPad2d 和 nn.ReplicationPad2d 来实现。

**一维填充层（1D Padding Layer）：**

 - 用于处理序列数据，如时间序列信号。

 - 在 PyTorch 中，可以使用 nn.ZeroPad1d、nn.ReflectionPad1d 和 nn.ReplicationPad1d 来实现。

**三维填充层（3D Padding Layer）：**

 - 用于处理视频数据或三维体积数据。

 - 在 PyTorch 中，可以使用 nn.ZeroPad3d、nn.ReflectionPad3d 和 nn.ReplicationPad3d 来实现。

```python
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.pool = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

net = Net()
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
net.to(device)
summary(net, (3, 32, 32))  # 查看模型结构
```

### Linear Layers
线性层（Linear Layers）在 PyTorch 中主要包括 nn.Linear、nn.Bilinear、nn.Identity 和 nn.LazyLinear。

 - nn.Linear 是最常用的线性层，用于实现全连接层的操作。它接受一个输入向量，并通过一个线性变换（即矩阵乘法）产生一个输出向量。如果需要，还可以加上一个偏置向量。
 - nn.Bilinear 是一个双线性层，它接受两个输入向量，并通过一个双线性变换产生一个输出向量。这个变换涉及两个输入向量与一个权重张量的乘法。
 - nn.Identity 是一个特殊的层，它不执行任何操作，只是直接返回输入数据。这个层主要用于构建模型时需要保持一致的接口，但又不需要实际的操作。
 - nn.LazyLinear 是一个延迟初始化的线性层，它允许在初始化时不指定输入特征的数量。输入特征的数量会在第一次前向传播时根据输入数据自动确定。

```python
class FNN(nn.Module):
    def __init__(self, input_dim, hidden_dim, output_dim):
        super(FNN, self).__init__()
        self.fc1 = nn.Linear(input_dim, hidden_dim)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        return x
```

### Normaliation Layers
归一化层（Normalization Layers）在深度学习中用于对输入数据进行归一化处理，以减少内部协变量移位（Internal Covariate Shift），提高训练速度并改善模型性能。PyTorch 提供了多种归一化层，包括 nn.BatchNorm1d、nn.BatchNorm2d、nn.BatchNorm3d、nn.InstanceNorm1d、nn.InstanceNorm2d、nn.InstanceNorm3d、nn.LayerNorm 和 nn.GroupNorm 等。
#### 归一化层的基本概念
**Batch Normalization：**

 - 在每个 mini-batch 上对数据进行归一化。
 - 通常用于加速训练过程和提高模型的泛化能力。

**Instance Normalization：**

 - 在每个样本上对数据进行归一化。

 - 通常用于图像生成任务，如风格迁移。

**Layer Normalization：**

 - 在每个样本的特征维度上进行归一化。

 - 通常用于循环神经网络（RNNs）和注意力机制（Attention Mechanisms）。

**Group Normalization：**

 - 将特征通道分成多个组，并在每个组内进行归一化。

 - 通常用于处理小批量数据。

#### 常见的归一化层类型
**Batch Normalization：**

```python
一维：nn.BatchNorm1d
二维：nn.BatchNorm2d
三维：nn.BatchNorm3d
```

**Instance Normalization：**

```python
一维：nn.InstanceNorm1d
二维：nn.InstanceNorm2d
三维：nn.InstanceNorm3d
```

**Layer Normalization：**

```python
nn.LayerNorm
```

**Group Normalization：**

```python
nn.GroupNorm
```
#### Batch Normalization、Instance Normalization、Layer Normalization 和 Group Normalization 的优点和缺点
**Batch Normalization**
**优点**

 - 减少内部协变量移位：通过在每个 mini-batch 上对数据进行归一化，可以减少训练过程中各层输入分布的变化。
 - 加速训练：归一化可以加快训练速度，使模型更快收敛。
 - 提高模型性能：归一化有助于提高模型的泛化能力。
 - 减少对正则化的需求：由于 Batch Normalization 具有一定的正则化效果，因此可以减少对 dropout 等其他正则化技术的需求。

**缺点**

 - 依赖 mini-batch 大小：Batch Normalization 的效果依赖于 mini-batch 的大小，较小的
   mini-batch 可能会导致不稳定的结果。
 - 增加计算开销：Batch Normalization 需要在每个 mini-batch 上计算均值和方差，增加了计算负担。
 - 影响模型的不确定性估计：在某些任务中，如贝叶斯深度学习，Batch Normalization 可能会影响模型的不确定性估计。

**Instance Normalization**
**优点**

 - 适用于图像生成任务：Instance Normalization 在图像生成任务中表现出色，特别是在风格迁移等任务中。

 - 简化模型设计：由于 Instance Normalization 不依赖于 mini-batch，因此在设计模型时更加灵活。 缺点

 - 不适合分类任务：Instance Normalization 可能在分类任务中的表现不如 Batch Normalization。

 - 不能处理 mini-batch 内的统计信息：Instance Normalization 对每个样本独立进行归一化，因此不能利用 mini-batch 内的统计信息。

**Layer Normalization**
**优点**

 - 适用于序列数据：Layer Normalization 在处理序列数据（如 RNNs）时非常有效。

 - 不受 mini-batch 大小的影响：Layer Normalization 不依赖于 mini-batch大小，因此在小批量训练时仍然有效。

**缺点**

 - 计算成本较高：Layer Normalization 需要在每个样本的特征维度上进行归一化，这可能会增加计算成本。
 - 可能不适合所有类型的模型：Layer Normalization 可能不是所有模型的最佳选择，特别是对于那些需要利用 mini-batch内统计信息的模型。

**Group Normalization**
**优点**

 - 适合小批量训练：Group Normalization 在小批量训练时表现良好，因为它将特征通道分成多个组进行归一化。

 - 灵活性高：可以通过调整组的数量来适应不同的模型和任务需求。

**缺点**

 - 计算成本：Group Normalization 相比于 Batch Normalization 可能会增加一些计算成本。
 - 参数调整：需要仔细选择组的数量以获得最佳性能。


```python
torch.nn.BatchNorm2d(num_features, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True, device=None, dtype=None)
```
torch.nn.BatchNorm2d 是 PyTorch 中用于二维卷积网络的批量归一化（Batch Normalization）层。它通过对每个 mini-batch 的数据进行归一化来减少内部协变量移位（Internal Covariate Shift），从而加速训练过程并提高模型的泛化能力。

**参数说明**
**num_features：**

 - 输入数据的特征数量（通常是通道数）。

**eps：**

 - 用于数值稳定性的微小值，默认为 1e-5。

**momentum：**

 - 更新统计量的动量值，默认为 0.1。

**affine：**

 - 是否使用可学习的仿射参数，默认为 True。

**track_running_stats：**

 - 是否跟踪运行均值和方差，默认为 True。

### Dropout Layers

Dropout 层是一种常用的正则化技术，在深度学习中用于防止过拟合。Dropout 通过随机地“丢弃”一部分神经元的输出，降低神经元之间的相互依赖性，从而提高模型的泛化能力。PyTorch 提供了多种 Dropout 层，包括 nn.Dropout、nn.Dropout2d 和 nn.Dropout3d 等。

**nn.Dropout** 是最基本的 Dropout 层，它随机地将输入张量中的元素设置为 0，而其他未被丢弃的元素则按比例放大。这种操作在训练时进行，而在评估时则不应用 Dropout。
**参数说明**

 - p：要丢弃的概率，默认为 0.5。
 - inplace：是否在原地修改输入，默认为 False。

### Alpha Dropout 
Alpha Dropout 是一种改进版的 Dropout 技术，专门设计用于解决激活函数为负值的问题，尤其是在使用带有负输出的激活函数（如 Leaky ReLU 或 Parametric ReLU）时更为适用。Alpha Dropout 通过保留负值输出的同时随机丢弃部分正输出，从而保持输入数据的均值和方差不变，有助于维持网络的稳定性。
#### Alpha Dropout 的工作原理
Alpha Dropout 的关键在于保持输入数据的均值和方差不变。在训练过程中，Alpha Dropout 会随机地将一部分正输出设置为 0，同时保留负输出。为了保持均值和方差不变，Alpha Dropout 还会对保留下来的输出进行缩放和平移。
**Alpha Dropout 通过以下步骤实现：**

 - 确定丢弃概率：基于激活函数的性质和期望的均值和方差，计算出丢弃概率。
 - 随机丢弃正输出：根据丢弃概率随机地将一部分正输出设置为 0。
 - 缩放和平移：为了保持均值和方差不变，对保留下来的输出进行缩放和平移。

**PyTorch 中的 nn.AlphaDropout**
PyTorch 提供了 nn.AlphaDropout 层来实现 Alpha Dropout。与标准的 Dropout 层相比，nn.AlphaDropout 更适用于使用 Leaky ReLU 或 Parametric ReLU 等激活函数的情况。

### Non-linear Layers

非线性层（Non-linear Layers）在深度学习中扮演着至关重要的角色，它们负责引入非线性变换到神经网络中，使得模型能够学习更复杂的函数映射。在 PyTorch 中，提供了多种非线性激活函数层，这些层通常位于线性层（如卷积层或全连接层）之后，用于增加模型的表达能力和学习能力。

#### 常见的非线性激活函数

1. **ReLU (Rectified Linear Unit)**
**优点：**
计算简单快速。
解决了梯度消失问题，特别是在深层网络中。
有助于加速训练过程。
由于稀疏激活，减少了计算资源的消耗。
**缺点：**
“死亡神经元”问题：当输入为负时，ReLU 的梯度为 0，这可能导致一些神经元永远不再激活。
输出不是均值为 0 的，可能会影响后续层的学习。
2. **Leaky ReLU**
**优点：**
解决了 ReLU 的“死亡神经元”问题，因为即使输入为负，也有非零梯度。
保持了 ReLU 的大部分优点，如计算效率和梯度消失问题的缓解。
**缺点：**
需要调整负斜率参数 (\alpha)，这增加了模型的超参数数量。
输出仍然不是均值为 0 的。
3. **PReLU (Parametric ReLU)**
**优点：**
具有 Leaky ReLU 的所有优点，并且 (\alpha) 作为可学习参数，可以自适应地调整。
更灵活，可以更好地适应不同任务的需求。
**缺点：**
需要额外的学习参数，可能会导致模型更加复杂。
计算成本略高于 ReLU。
4. **ELU (Exponential Linear Units)**
**优点：**
在输入为负时，ELU 的输出接近 0，这有助于保持均值接近 0，有利于梯度传播。
可以缓解梯度消失问题，因为梯度永远不会为 0。
**缺点：**
当输入为负时，计算成本较高，因为涉及到指数运算。
需要调整 (\alpha) 参数。
5. **SELU (Scaled Exponential Linear Units)**
**优点：**
自动校准和稳定网络的均值和方差，有助于消除批量归一化的需要。
可以保证网络的自归一化属性，即网络的输出保持一定的均值和方差。
**缺点：**
计算成本较高，特别是当输入为负时。
需要特定的初始化方法才能发挥最佳效果。
6. **Sigmoid**
**优点：**
输出范围为 [0, 1]，适合用于二分类任务。
计算简单。
**缺点：**
容易导致梯度消失问题，特别是在深层网络中。
输出不是均值为 0 的，可能会影响后续层的学习。
7. **Tanh (Hyperbolic Tangent)**
**优点：**
输出范围为 [-1, 1]，有助于保持均值接近 0。
计算简单。
**缺点：**
同样容易导致梯度消失问题，特别是在深层网络中。
输出不是均值为 0 的，可能会影响后续层的学习。
8. **Softmax**
**优点：**
适用于多分类任务，输出为概率分布。
输出之和为 1，便于解释。
**缺点：**
仅适用于多分类任务，不适合用于其他类型的网络层。
在计算时可能会遇到数值稳定性问题，需要特殊处理。

在 PyTorch 中，这些非线性激活函数可以通过 torch.nn 模块中的相应类来实现。

```python
Containers： 模型容器
Convolution Layers：卷积层
Pooling layers：池化层
Padding Layers：填充层
Non-linear Activations (weighted sum, nonlinearity)：非线性激活函数
Non-linear Activations (other)：Softmax系列激活函数
Normalization Layers：标准化层
Recurrent Layers：RNN 网络层
Transformer Layers： Transformer 网络层
Linear Layers：线性层
Dropout Layers： 随机失活层
Sparse Layers：稀疏网络层
Distance Functions：计算距离函数
Loss Functions：计算损失函数
Vision Layers：CV任务网络层
Shuffle Layers：随机打乱功能层
DataParallel Layers (multi-GPU, distributed)：多GPU网络层，多gpu需要用层的概念进行包装
Utilities：各功能函数层
Quantized Functions：量化功能函数
Lazy Modules Initialization：“懒惰”初始化功能模块
```
## Module常用API函数

### 设置模型训练、评估模式
设置模型的训练模式和评估模式，调用 model.train() 和 model.eval() 方法即可。
#### 训练模式
训练模式通常用于模型的训练阶段，在这个模式下，模型中的某些层（如 Batch Normalization 和 Dropout）会有不同的行为。

 - Batch Normalization：在训练模式下，Batch Normalization会计算每一批次的均值和方差，并用这些统计信息来标准化输入数据。
 - Dropout：在训练模式下，Dropout 会随机丢弃一部分节点，以减少过拟合。
#### 评估模式
评估模式通常用于模型的验证或测试阶段，在这个模式下，模型中的某些层（如 Batch Normalization 和 Dropout）会有不同的行为。
 - Batch Normalization：在评估模式下，Batch Normalization会使用整个训练集的均值和方差来标准化输入数据。
 - Dropout：在评估模式下，Dropout 不再随机丢弃节点，而是将所有节点的输出乘以丢弃概率，以模拟训练时的行为。

### 设置模型存放在cpu/gpu
在 PyTorch 中，可以轻松地将模型设置为在 CPU 或 GPU 上运行。这通常是通过 .to() 方法完成的，该方法允许你指定模型运行的设备。

```python
import torch
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self):
        super(SimpleNet, self).__init__()
        self.fc1 = nn.Linear(10, 10)
        self.fc2 = nn.Linear(10, 5)

    def forward(self, x):
        x = self.fc1(x)
        x = self.fc2(x)
        return x

# 创建模型实例
model = SimpleNet()

# 获取模型的所有参数
params = model.parameters()

# 打印每个参数的形状
for param in params:
    print(param.shape)

# 获取模型的所有参数及其名称
named_params = model.named_parameters()

# 打印每个参数的名称和形状
for name, param in named_params:
    print(name, param.shape)

# 保存模型权重
torch.save(model.state_dict(), 'model_weights.pth')

# 创建一个新的模型实例
new_model = SimpleNet()

# 加载权重
new_model.load_state_dict(torch.load('model_weights.pth'))

# 将新模型移动到选定的设备上
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
new_model.to(device)

# 创建输入数据
input_data = torch.randn(1, 10).to(device)

# 前向传播
output = new_model(input_data)
print("Output:", output)
```

### 获取模型参数、加载权重参数
模型训练完毕后，需要保存的核心内容是模型参数，这样可以供下次使用，或者是给别人进行finetune。
在 PyTorch 中，state_dict 和 load_state_dict 用于保存和加载模型的参数。
1. state_dict
state_dict 是一个 Python 字典对象，它包含了模型的所有可学习参数（权重和偏置）。键是模型中每一层的名称，值是对应的 Tensor 对象。当你调用 model.state_dict() 时，会得到一个这样的字典，其中包含了模型的所有参数。
2. load_state_dict
load_state_dict 方法用于将保存的状态字典加载回模型中。你需要确保新模型的结构与保存状态字典时的模型结构完全一致。否则，加载时可能会出现问题。

### 管理模型的modules, parameters, sub_module
在 PyTorch 中，管理模型的模块（modules）、参数（parameters）和子模块（sub-modules）是构建和操作神经网络的重要方面。
1. 管理 Modules
在 PyTorch 中，nn.Module 是所有神经网络模块的基础类。当您创建自定义模型时，通常会继承 nn.Module 并在其构造函数中添加其他模块。这些模块会被自动跟踪并存储在 _modules 字典中。
访问 Modules
您可以使用 modules() 或 named_modules() 方法来访问模型中的所有模块。
2. 管理 Parameters
模型的参数（例如权重和偏置）可以通过 parameters() 或 named_parameters() 方法来访问。
3. 管理 Sub-modules
子模块是指模型内部的其他 nn.Module 实例。您可以使用 children() 或 named_children() 方法来访问模型的直接子模块。
### 设置模型的参数精度，可选半精度、单精度、双精度等

在 PyTorch 中，设置模型参数的精度（如半精度、单精度或双精度）对于优化内存使用和提高计算效率非常重要。
1. 半精度 (Half Precision, float16)
半精度浮点数（float16）可以显著减少内存使用量，适用于 GPU 上的训练和推理。但是，由于其精度较低，可能会影响模型的收敛性和最终性能。
2. 单精度 (Single Precision, float32)
单精度浮点数（float32）是默认的精度设置，适用于大多数情况下的训练和推理。
设置单精度
通常情况下，模型默认就是单精度的，无需特别设置。
3. 双精度 (Double Precision, float64)
双精度浮点数（float64）提供了更高的精度，适用于需要更高精度的场景，如科学计算。但是，它会占用更多的内存。

### 对子模块执行特定功能
1. zero_grad
zero_grad 方法用于将模型的所有参数的梯度设置为 0 或 None。这对于每次训练迭代开始之前清空梯度非常重要，避免梯度累积导致错误的结果。
2. apply
apply 方法用于对模型的所有子模块执行指定的函数。这在参数初始化、模型修改等方面非常有用。

```python
import torch
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self):
        super(SimpleNet, self).__init__()
        self.fc1 = nn.Linear(10, 10)
        self.fc2 = nn.Linear(10, 5)

    def forward(self, x):
        x = self.fc1(x)
        x = self.fc2(x)
        return x

# 创建模型实例
model = SimpleNet()

# 清空模型参数的梯度
model.zero_grad()

# 定义初始化线性层权重的函数
def initialize_linear_layers(module, init_value=0.1):
    """
    初始化 Linear 层的权重。
    
    :param module: 当前模块
    :param init_value: 初始化值
    """
    if isinstance(module, nn.Linear):
        # 初始化权重
        nn.init.constant_(module.weight, init_value)
        # 如果存在偏置，则初始化偏置
        if module.bias is not None:
            nn.init.constant_(module.bias, 0.0)

# 对所有子模块执行初始化函数
model.apply(initialize_linear_layers)

# 打印模型的参数以验证初始化
for name, param in model.named_parameters():
    print(name, param)
```

## Hook函数及Grad
在 PyTorch 中，Hook 是一种机制，允许用户在模型的前向传播和反向传播过程中拦截模块的输入或输出，并执行自定义的操作。Hook 可以用来监听特定模块的行为，比如打印输入输出的形状、修改输入输出的数据、记录梯度等，这对于调试模型、可视化中间结果、修改模型行为等都非常有用。

#### torch.Tensor.register_hook：

 - 用途：监听特定张量的梯度。
 - 位置：反向传播过程中的梯度计算。
 - 触发条件：当计算该张量的梯度时触发。
 - 参数：梯度 (grad)。

#### torch.nn.Module.register_forward_hook：

 - 用途：监听模块的输出。 
 - 位置：前向传播过程中的输出。 
 - 触发条件：当模块的前向传播完成时触发。 
 - 参数：模块 (module)、输入 (input) 和输出 (output)。

#### torch.nn.Module.register_forward_pre_hook：

 - 用途：监听模块的输入。
 -  位置：前向传播过程中的输入。
 -  触发条件：当模块的前向传播即将开始时触发。 
 - 参数：模块 (module)、输入 (input)。

#### torch.nn.Module.register_full_backward_hook：

 - 用途：监听模块的输入和输出梯度。 
 - 位置：反向传播过程中的梯度计算。 
 - 触发条件：当计算该模块的输入和输出梯度时触发。 
 - 参数：模块  (module)、输入梯度 (grad_input) 和输出梯度 (grad_output)。

### Grad-CAM

Grad-CAM（Gradient-weighted Class Activation Mapping）是一种用于可视化卷积神经网络（CNN）中哪些区域对于分类决策最为重要的技术。它通过计算特征图上的梯度来生成热力图，从而高亮显示图像中对预测类别贡献最大的部分。
Grad-CAM 的实现步骤

 1. 选择目标层：通常选择靠近输出的卷积层作为目标层。 
 2. 获取特征图：在前向传播过程中获取目标层的特征图。
 3. 计算梯度：在反向传播过程中计算目标类别的梯度。 
 4. 计算权重：根据梯度计算每个通道的权重。 
 5. 生成热力图：使用权重加权平均特征图，得到热力图。
 6. 叠加到原始图像：将热力图与原始图像叠加，可视化哪些区域对分类贡献最大。

## 权重初始化方法
权重初始化是深度学习模型训练中的一个重要环节，良好的初始化方法能够帮助模型更快地收敛，并且减少陷入局部最优的风险。下面列举了一些常见的权重初始化方法，并解释它们的作用和适用场景。在 PyTorch 中，可以通过多种方式来初始化权重。例如，可以使用 nn.init 模块提供的函数来初始化模型的参数。
### 常见的权重初始化方法
#### 零初始化 (Zero Initialization)

 - 描述：将所有权重设置为0。
 - 缺点：会导致隐藏层的对称性问题，即所有神经元在每次迭代中更新相同的权重，无法学习到不同的特征。
 - 适用场景：不推荐用于多层神经网络，但在某些情况下可用于线性模型或逻辑回归。
#### 随机初始化 (Random Initialization)
 - 描述：将权重初始化为小的随机数，通常是均匀分布或正态分布。 
 - 优点：打破对称性，使每个神经元从不同的起点开始学习。
 - 适用场景：广泛应用于多层神经网络。

#### Xavier/Glorot 初始化 (Xavier/Glorot Initialization) 
 - 优点：有助于保持各层的输出方差大致相同，减少梯度消失或爆炸的问题。 
 - 适用场景：适用于使用 sigmoid 或 tanh 激活函数的模型。

#### He 初始化 (He Initialization)

 - 优点：特别设计用于 ReLU 或其他非线性激活函数，有助于缓解梯度消失问题。 
 - 适用场景：适用于使用 ReLU 或其变体（如 Leaky ReLU）作为激活函数的模型。

#### 正交初始化 (Orthogonal Initialization)

 - 描述：通过构造正交矩阵来初始化权重。 
 - 优点：有助于保持各层的输出方差大致相同，减少梯度消失或爆炸的问题。 
 - 适用场景：适用于 RNNs 和其他递归结构。

#### 预训练初始化 (Pre-trained Initialization)

 - 描述：使用预先训练好的模型的权重作为初始化。 
 - 优点：可以利用预训练模型学到的特征，加速训练过程。
 - 适用场景：迁移学习任务，特别是当目标数据集较小或领域相似时。

## 示例

```python
import torch

class EarlyStopping:
    def __init__(self, patience=10, delta=0, filepath='checkpoint.pth'):
        """
        Early stops the training if validation loss doesn't improve after a given patience.
        
        Args:
            patience (int): How long to wait after last time validation loss improved.
                            Default: 10
            delta (float): Minimum change in the monitored quantity to qualify as an improvement.
                            Default: 0
            filepath (str): Path to save the best model checkpoint.
                            Default: 'checkpoint.pth'
        """
        self.patience = patience
        self.delta = delta
        self.filepath = filepath
        self.counter = 0
        self.best_score = None
        self.early_stop = False

    def __call__(self, val_loss, model):
        score = -val_loss
        
        if self.best_score is None:
            self.best_score = score
            self._save_checkpoint(model)
        elif score < self.best_score + self.delta:
            self.counter += 1
            if self.counter >= self.patience:
                self.early_stop = True
        else:
            self.best_score = score
            self._save_checkpoint(model)
            self.counter = 0

    def _save_checkpoint(self, model):
        torch.save(model.state_dict(), self.filepath)

    def load_best_model(self, model):
        model.load_state_dict(torch.load(self.filepath))

def optuna _search_dl(train_loader, val_loader, model):

	def objective(trial):
		# 定义超参数搜索空间
		learning_rate = trial,suggest_float('learning_rate', 1e-5, 1e-2, log=True)
		# 构建型
		criterion = nn.L1Loss()
		# criterion = nn.MSELoss()
		optimizer = torch.optim.Adam(model.parameters(), lr=learning_rate)
		# optimizer =torch.optim.SGD(model.parameters()
		lr=learning rate, momentum=0.9)
		# 实例化 Earlystopping
		early_stopping = EarlyStopping(patience=10,delta=0)
		# 训练模型
		for epoch in range(100):
			model.train()
			total loss = 0
			num_samples =0
			
				for inputs, targets in train loader:
					inputs,targets = inputs.to(config.device),
					targets.to(config.device)
					optimizer.zero grad()
					outputs = model(inputs)
					loss = criterion(outputs,targets)
					Loss.backward()
					optimizer.step()
					total loss += loss.item()* inputs.size(0)
					num_samples += inputs.size(0)
			train loss =total loss /num_samples
		# 验证集评估
		val loss =evaluate_model(model, val loader, criterion)
		if early_stopping(val loss, model):
			break
			
		if(epoch + 1)% 10 == 0:
			print(f"Epoch: fepoch+1}, Train Loss: {train_loss:.4f}, Val Loss: {val_loss:.4f}")
			
		# 返回验证集损失作为目标函数值
		return early_stopping.val loss_min
		
	study =optuna.create_study(direction='minimize')
	study.optimize(objective,n trials=16)
	dl params =study.best trial.params
	print(dl_params)
	return dl params
```
### RNN模型

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision.datasets import MNIST
import torchvision.transforms as transforms

# Set hyperparameters
batch_size = 64
input_size = 28  # MNIST images are 28x28 pixels
hidden_size = 128
num_layers = 2
num_classes = 10
num_epochs = 10
learning_rate = 0.001

# Load the MNIST dataset
transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize((0.1307,), (0.3081,))])
trainset = MNIST(root='./data', train=True, download=True, transform=transform)
testset = MNIST(root='./data', train=False, download=True, transform=transform)

# Create data loaders
trainloader = DataLoader(trainset, batch_size=batch_size, shuffle=True)
testloader = DataLoader(testset, batch_size=batch_size, shuffle=False)


# Define the RNN model
class RNN(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, num_classes):
        super(RNN, self).__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.rnn = nn.RNN(input_size, hidden_size, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_size, num_classes)

    def forward(self, x):
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        out, _ = self.rnn(x, h0)
        out = self.fc(out[:, -1, :])
        return out

# Initialize the model
model = RNN(input_size, hidden_size, num_layers, num_classes)


# Define loss function and optimizer
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=learning_rate)

# Training loop
total_step = len(trainloader)
for epoch in range(num_epochs):
    for i, (images, labels) in enumerate(trainloader):
        images = images.reshape(-1, input_size, input_size)

        # Forward pass
        outputs = model(images)
        loss = criterion(outputs, labels)

        # Backward and optimize
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        if (i + 1) % 100 == 0:
            print(f'Epoch [{epoch+1}/{num_epochs}], Step [{i+1}/{total_step}], Loss: {loss.item():.4f}')

# Testing
model.eval()

with torch.no_grad():
    correct = 0
    total = 0
    for images, labels in testloader:
        images = images.reshape(-1, input_size, input_size)
        outputs = model(images)
        _, predicted = torch.max(outputs.data, 1)
        total += labels.size(0)
        correct += (predicted == labels).sum().item()

    print(f'Test Accuracy: {100 * correct / total:.2f}%')
```
### TCN模型
TCN（Temporal Convolutional Network，时域卷积网络）是一种用于处理序列数据的深度学习模型，它利用一维卷积层来捕捉时间序列中的模式。
#### TCN的基本结构
**因果卷积**：为了确保网络只能访问当前时刻及之前的数据，TCN使用了因果卷积（causal convolution）。这意味着每个卷积核只考虑当前时间步及其之前的输入值。
**扩张卷积**（Dilated Convolutions）：通过使用不同扩张率（dilation rate）的卷积核，TCN可以在保持参数数量不变的情况下捕获不同时间尺度上的依赖关系。扩张卷积允许模型接收更大范围的历史信息而不增加网络的深度或宽度。
**残差连接**：类似于ResNet中的残差块，TCN也使用残差连接来帮助梯度流过更深层的网络，从而缓解梯度消失问题。
#### TCN的优势
**并行化**：与递归神经网络（如LSTM或GRU）相比，TCN可以更容易地实现并行化，因为卷积操作可以在整个输入序列上同时执行。
**长期依赖性**：由于扩张卷积的存在，TCN能够有效地捕捉长距离的依赖关系。
**避免梯度消失/爆炸**：卷积层通常不会像递归层那样容易遇到梯度消失或爆炸的问题。

```python
class Chomp1d(nn.Module):
    def __init__(self, chomp_size):
        super(Chomp1d, self).__init__()
        self.chomp_size = chomp_size

    def forward(self, x):
        return x[:, :, :-self.chomp_size].contiguous()

class TemporalBlock(nn.Module):
    def __init__(self, n_inputs, n_outputs, kernel_size, stride, dilation, padding, dropout=0.2):
        super(TemporalBlock, self).__init__()
        self.conv1 = nn.utils.weight_norm(nn.Conv1d(n_inputs, n_outputs, kernel_size,
                                                     stride=stride, padding=padding, dilation=dilation))
        self.chomp1 = Chomp1d(padding)
        self.relu1 = nn.ReLU()
        self.dropout1 = nn.Dropout(dropout)

        self.conv2 = nn.utils.weight_norm(nn.Conv1d(n_outputs, n_outputs, kernel_size,
                                                     stride=stride, padding=padding, dilation=dilation))
        self.chomp2 = Chomp1d(padding)
        self.relu2 = nn.ReLU()
        self.dropout2 = nn.Dropout(dropout)

        self.net = nn.Sequential(self.conv1, self.chomp1, self.relu1, self.dropout1,
                                 self.conv2, self.chomp2, self.relu2, self.dropout2)
        self.downsample = nn.Conv1d(n_inputs, n_outputs, 1) if n_inputs != n_outputs else None
        self.relu = nn.ReLU()
        self.init_weights()

    def init_weights(self):
        self.conv1.weight.data.normal_(0, 0.01)
        self.conv2.weight.data.normal_(0, 0.01)
        if self.downsample is not None:
            self.downsample.weight.data.normal_(0, 0.01)

    def forward(self, x):
        out = self.net(x)
        res = x if self.downsample is None else self.downsample(x)
        return self.relu(out + res)


class TemporalConvNet(nn.Module):
    def __init__(self, num_inputs, num_channels, kernel_size=2, dropout=0.2):
        super(TemporalConvNet, self).__init__()
        layers = []
        num_levels = len(num_channels)
        for i in range(num_levels):
            dilation_size = 2 ** i
            in_channels = num_inputs if i == 0 else num_channels[i-1]
            out_channels = num_channels[i]
            layers += [TemporalBlock(in_channels, out_channels, kernel_size, stride=1, dilation=dilation_size,
                                     padding=(kernel_size-1) * dilation_size, dropout=dropout)]

        self.network = nn.Sequential(*layers)

    def forward(self, x):
        return self.network(x)

```
### Transformer模型
![在这里插入图片描述](/img/a3e4aa1e58f64f6784e36d2764269a85.png)

Transformer是自然语言处理（NLP）领域的一个重要模型，它改变了传统的基于循环神经网络（RNN）的序列建模方式，引入了自注意力机制（self-attention mechanism），使得模型能够并行处理输入数据，并且更好地捕捉长距离依赖关系。

 - **自注意力机制（Self-Attention）**：允许模型中的每个位置直接关注到序列中的所有位置，从而捕捉全局信息。
 - **多头注意力（Multi-Head Attention）**：将注意力机制分成多个“头”，每个头可以独立地关注不同的信息，然后将这些信息合并起来。
 - **位置编码（Positional Encoding）**：由于自注意力机制并不知道序列中元素的位置信息，因此需要额外的位置编码来提供这一信息。
 - **编码器-解码器结构**：Transformer由多个编码器（Encoder）层和解码器（Decoder）层组成。编码器层负责处理输入序列，解码器层负责生成输出序列。
 - **残差连接与层规范化（Residual Connections and Layer Normalization）**：为了帮助梯度流动和稳定训练，每个子层前后都加入了残差连接，并且在每个子层之后应用了层规范化。

```python
class TransformerPredictor(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, num_heads, dropout):
        super(PowerPredictor, self).__init__()
        self.input_size = input_size
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.num_heads = num_heads
        self.dropout = dropout

        self.embedding = nn.Linear(input_size, hidden_size)
        encoder_layer = TransformerEncoderLayer(d_model=hidden_size, nhead=num_heads, dim_feedforward=hidden_size * 4, dropout=dropout)
        self.transformer_encoder = TransformerEncoder(encoder_layer, num_layers)
        self.fc = nn.Linear(hidden_size, 1)

    def forward(self, x):
        x = self.embedding(x)
        x = x.permute(1, 0, 2)  # 调整维度顺序为 (seq_len, batch_size, hidden_size)
        x = self.transformer_encoder(x)
        x = x.permute(1, 0, 2)  # 调整回 (batch_size, seq_len, hidden_size)
        x = x[:, -1, :]  # 取最后一个时间步的输出
        x = self.fc(x)
        return x
```
#### nn.TransformerEncoderLayer
nn.TransformerEncoderLayer 包含以下几个关键部分：

**Multi-head Self-Attention (MHSA)**:

 - 这是 Transformer 编码器层中最核心的部分之一。它允许模型关注输入的不同位置，从而捕捉不同位置之间的依赖关系。
 - Multi-head自注意力机制将输入分成多个头，在不同的表示子空间中并行执行注意力计算，然后将它们连接起来并通过一个线性层投影回原来的维度。

**Position-wise Feed-Forward Networks (FFN):**

 - FFN 是一个两层的全连接前馈网络，它对序列中的每个位置单独且并行地应用同样的线性变换。

 - 这个网络由两个线性层组成，中间夹着一个 ReLU 或 GELU 激活函数。

**Layer Normalization:**

 - 在 MHSA 和 FFN 之后都会应用 LayerNorm 层，以帮助稳定训练过程。

**Residual Connections:**

 - 每个子层（MHSA 和 FFN）周围都有残差连接，这意味着子层的输入被直接加到子层的输出上。

**参数说明**

```python
d_model: 输入的特征维度。这也是模型内部的所有层的特征维度。
nhead: 多头注意力机制中的头数。每个头独立地关注输入的不同部分。
dim_feedforward: 前馈网络中线性层的隐藏维度。
dropout: 应用于各个子层的 dropout 比率。
activation: 前馈网络中的激活函数，默认通常是 ReLU 或 GELU。
```
#### nn.TransformerEncoder 的结构
nn.TransformerEncoder 由多个 nn.TransformerEncoderLayer 组成，每个层都包含了多头自注意力机制（Multi-Head Self-Attention）和位置前馈网络（Position-wise Feed-Forward Network）。这些层之间通过残差连接（Residual Connections）和层规范化（Layer Normalization）连接在一起。
**主要组成部分**
**Multi-Head Self-Attention (MHSA):**

 - 这是编码器层的核心部分，它使得模型能够在输入序列的不同位置之间建立联系。 每个“头”都独立地执行自注意力计算，允许模型关注输入的不同方面。
 - 所有头的结果被拼接起来，并通过一个线性层映射回原来的维度。

**Position-wise Feed-Forward Networks (FFNs):**

 - 这是一个简单的两层前馈神经网络，它对序列中的每个位置分别应用相同的线性变换。

 - FFN 有助于捕捉更复杂的模式，并且可以在不同的位置上并行计算。

**Layer Normalization:**

 - 在 MHSA 和 FFN 之后都会应用 LayerNorm，以加速训练并提高模型的稳定性。

**Residual Connections:**

 - 每个子层（MHSA 和 FFN）的输入都被加到子层的输出上，形成残差连接。

参考：[https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-4/](https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-4/)
参考：[https://pytorch-cn.readthedocs.io/zh/latest/](https://pytorch-cn.readthedocs.io/zh/latest/)
参考：[https://datawhalechina.github.io/thorough-pytorch/](https://datawhalechina.github.io/thorough-pytorch/)
参考：[https://datawhalechina.github.io/grape-book/#/docs\03%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E5%9F%BA%E7%A1%80\03%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E5%9F%BA%E7%A1%80?id=_31-%e7%a5%9e%e7%bb%8f%e7%bd%91%e7%bb%9c%e5%8f%8a%e5%85%b6%e5%9f%ba%e6%9c%ac%e7%bb%84%e6%88%90](https://datawhalechina.github.io/grape-book/#/docs%5C03%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E5%9F%BA%E7%A1%80%5C03%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E5%9F%BA%E7%A1%80?id=_31-%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C%E5%8F%8A%E5%85%B6%E5%9F%BA%E6%9C%AC%E7%BB%84%E6%88%90)
