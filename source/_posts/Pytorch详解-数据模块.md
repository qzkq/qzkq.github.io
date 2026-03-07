---
title: Pytorch详解-数据模块
date: 2026-01-03 08:05:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Pytorch详解-数据模块)

**Dataset**是一个抽象基类，提供给用户定义自己的数据读取方式，用户可以通过继承 Dataset 类并实现其方法来定制数据加载逻辑。其中最核心的方法是 __getitem__，它定义了如何根据索引获取单个样本。

```python
__getitem__(self, index):
```

 - 这个方法接收一个整数 index 作为输入。
 - 根据 index 加载并返回对应的数据样本。
 - 数据样本通常包括特征和标签，可以是任何格式，如图像、文本、音频等。
 - 返回值通常是元组 (sample, target)，其中 sample 是特征数据，target 是对应的标签。

**DataLoader**是pytorch数据加载的核心，其中包括多个功能，如打乱数据，采样机制（实现均衡1:1采样），多进程数据加载，组装成Batch形式等丰富的功能。
## torch.utils.data.Dataset
### 数据交互模块—Dataset的功能

在Dataset类的编写中必须要实现的两个函数是`__getitem__`和`__len__`。

 - **getitem**：需要实现读取一个样本的功能。通常是传入索引（index，可以是序号或key），然后实现从磁盘中读取数据，并进行预处理（包括online的数据增强），然后返回一个样本的数据。数据可以是包括模型需要的输入、标签，也可以是其他元信息，例如图片的路径。getitem返回的数据会在dataloader中组装成一个batch。即，通常情况下是在dataloader中调用Dataset的getitem函数获取一个样本。
 - **len**：返回数据集的大小，数据集的大小也是个最要的信息，它在dataloader中也会用到。如果这个函数返回的是0，dataloader会报错："ValueError: num_samples should be a positive integer value, but got num_samples=0"
这个报错相信大家经常会遇到，这通常是文件路径没写对，导致你的dataset找不到数据，数据个数为0。
 - **\__init__ 方法**：初始化数据集，通常在这里进行数据集的预处理或加载。

![在这里插入图片描述](/img/40ad79df5ee9432d9bf8e9e727ac9448.png)
dataset负责与磁盘打交道，将磁盘上的数据读取并预处理好，提供给DataLoader，而DataLoader只需要关心如何组装成批数据，以及如何采样。采样的体现是出现在传入getitem函数的索引，这里采样的规则可以通过sampler由用户自定义，可以方便地实现均衡采样、随机采样、有偏采样、渐进式采样等。
### 示例

```python
class PowerPredictionDataset(Dataset):
    def __init__(self, features, targets):
        self.features = torch.tensor(features.values, dtype=torch.float)
        self.targets = torch.tensor(targets.values, dtype=torch.float)

    def __len__(self):
        return len(self.features)

    def __getitem__(self, idx):
        return self.features[idx], self.targets[idx]
```

### 系列APIs
#### concat
在实际项目中，数据的来源往往是多源的，可能是多个中心收集的，也可能来自多个时间段的收集，很难将可用数据统一到一个数据形式。通常有两种做法，一种是固定一个数据形式，所有获取到的数据经过整理，变为统一格式，然后用一个Dataset即可读取。还有一种更为灵活的方式是为每批数据编写一个Dataset，然后使用`torch.utils.data.ConcatDataset`类将他们拼接起来，这种方法可以灵活的处理多源数据，也可以很好的使用别人的数据及Dataset。
**使用 ConcatDataset 的注意事项**

 - 索引: 当你从 ConcatDataset 中获取样本时，索引会跨越所有的子数据集。例如，如果 dataset1 有 50 个样本，dataset2 有 30 个样本，则 combined_dataset 将会有 80 个样本。索引 50 将指向 dataset2 中的第一个样本。
 - 数据一致性: 确保所有子数据集的样本具有相同的数据结构和类型。例如，如果 dataset1 的每个样本都是一个字典，那么 dataset2 也应该如此。
 - 转换: 如果你需要对数据应用转换（例如，图像增强），确保所有子数据集都使用相同的转换逻辑，或者在 ConcatDataset 外部处理转换。
#### Subset
用于从现有的 Dataset 中选取一部分样本，形成一个新的数据集。这对于分割训练集、验证集或测试集非常有用。
**Subset 类的关键方法**
 **\__init__** 方法

作用：初始化 Subset，指定基础数据集和要使用的索引。
参数：

 - dataset: 基础的 Dataset 对象。
 - indices: 一个整数列表，表示从基础数据集中选择的样本索引。

**\__getitem__** 方法
作用：根据索引从基础数据集中获取样本。
参数：

 - idx: 在 Subset 中的索引。

**\__len__** 方法
作用：返回 Subset 中的样本数量。
#### random_split
该函数的功能是随机的将dataset划分为多个不重叠的子集，适合用来划分训练、验证集（不过不建议通过它进行，因为对用户而言，其划分不可见，不利于分析）。

```python
torch.utils.data.random_split(dataset, lengths, generator=None)
```
参数:

 - dataset: 要分割的基础 Dataset 对象。
 - lengths: 一个整数列表，表示分割后每个子数据集的长度。
 - generator: 一个可选的 torch.Generator 对象，用于控制随机数生成器的状态，以确保结果的可重复性。

返回:
一个包含分割后子数据集的列表。
#### sampler
`torch.utils.data.Sampler` 是 PyTorch 中的一个抽象基类，用于定义从数据集中抽取样本的方式。主要是设置挑选策略，如按顺序挑选、随机挑选、按类别分概率挑选等等，这些都可以通过自定义sampler实现。

**Sampler 类的关键方法**

 - **\__iter__** 方法
作用：返回一个迭代器，该迭代器会产生数据集中的样本索引。
 - **\__len__** 方法
作用：返回迭代器产生的索引的数量。

**常用的 Sampler 类**

**RandomSampler:**
作用：随机抽取数据集中的样本索引。
参数：

 - data_source: 数据集。
 - replacement: 是否允许重复抽样，默认为 False。

**SequentialSampler:**
作用：按顺序抽取数据集中的样本索引。
参数：

 - data_source: 数据集。

**BatchSampler:**
作用：从另一个 Sampler 中抽取批次。
参数：

 - sampler: 基础的 Sampler。
 - batch_size: 每个批次的大小。
 - drop_last: 如果最后一个批次的大小小于 batch_size 是否丢弃，默认为 False。

**WeightedRandomSampler:**
作用：根据给定的权重随机抽取样本索引。
参数：

 - weights: 一个列表或张量，包含每个样本的权重。
 - num_samples: 抽取的样本数量。
 - replacement: 是否允许重复抽样，默认为 True。

**SubsetRandomSampler:**
作用：从给定的索引列表中随机抽取样本索引。
参数：

 - indices: 一个整数列表，包含要抽取的样本索引。

**DistributedSampler:**
作用：在分布式训练中使用，确保每个进程获得不同的数据子集。
参数：

 - dataset: 数据集。
 - num_replicas: 总的进程数量。
 - rank: 当前进程的编号。

### unsqueeze
在PyTorch中，unsqueeze 是一个用于增加张量维度的方法。
**功能：**
unsqueeze(dim) 方法会在指定的维度 dim 处插入一个新的维度（大小为1）。
**参数：**
dim (int) – 新的维度会被插入到这个位置上。
**示例：**
如果有一个形状为 (3, 4) 的张量，调用 .unsqueeze(0) 后，其形状会变为 (1, 3, 4)。如果调用 .unsqueeze(1)，则形状会变为 (3, 1, 4)。
这个方法非常有用，尤其是在调整张量的维度以满足某些操作或模型输入的要求时。
## DataLoader

```python
torch.utils.data.DataLoader(dataset, batch_size=1, shuffle=False, sampler=None, batch_sampler=None, num_workers=0, collate_fn=None, pin_memory=False, drop_last=False, timeout=0, worker_init_fn=None, multiprocessing_context=None, generator=None, *, prefetch_factor=2, persistent_workers=False)
```
参数:

```python
dataset: Dataset 类型的对象，包含要加载的数据。
batch_size: 每个批次的样本数量，默认为 1。
shuffle: 是否在每个 epoch 开始时打乱数据集，默认为 False。
sampler: Sampler 类型的对象，用于定义从数据集中抽取样本的方式。
batch_sampler: BatchSampler 类型的对象，用于定义从数据集中抽取批次的方式。
num_workers: 加载数据的子进程数量，默认为 0，表示在主线程中加载数据。
collate_fn: 一个函数，用于合并一批次内的样本数据。
pin_memory: 是否将数据复制到 CUDA 的固定内存中，以提高数据传输速度，默认为 False。
drop_last: 如果最后一个批次的大小小于 batch_size 是否丢弃，默认为 False。
timeout: 设置数据加载的超时时间（单位：秒）。
worker_init_fn: 一个函数，用于初始化每个子进程。
multiprocessing_context: 控制子进程的启动方式。
generator: 一个 torch.Generator 对象，用于控制随机数生成器的状态。
prefetch_factor: 每个工作进程提前加载的批次数量。
persistent_workers: 是否在数据加载完成后保留子进程，默认为 False。
```

### DataLoader功能
#### 支持两种形式数据集读取
1. 迭代器形式
在迭代器形式下，DataLoader 会自动遍历整个数据集，并按照指定的批次大小返回数据。这种方式适用于典型的训练循环，其中数据集会被多次遍历，每个 epoch 结束后数据集会被重新打乱（如果设置了 shuffle=True）。
2. 索引形式
在索引形式下，用户可以直接通过索引来获取特定批次的数据。这种方式适用于需要对数据集进行更细粒度控制的情况，例如在调试阶段或者需要手动控制数据流的情况。
#### 自定义采样策略

DataLoader可借助Sampler自定义采样策略，包括为每个类别设置采样权重以实现1:1的均衡采样，或者是自定义采样策略。
#### 自动组装成批数据

mini-batch形式的训练成为了深度学习的标配，如何把数据组装成一个batch数据？DataLoader内部自动实现了该功能，并且可以通过batch_sampler、collate_fn来自定义组装的策略，十分灵活。

如果你需要对数据进行特殊的批处理逻辑，可以通过定义 collate_fn 函数来实现。collate_fn 函数接收一个样本列表，并返回一个批次数据。例如，如果数据集中的每个样本是一个字典，你可以定义一个 collate_fn 函数来将这些字典合并成一个批次。

#### 多进程数据加载

通常GPU运算消耗数据会比CPU读取加载数据要快，CPU“生产”跟不上GPU“消费”，因此需要多进程进行加载数据，以满足GPU的消费需求。通常指要设置num_workers 为CPU核心数，如16核的CPU就设置为16。

#### 自动实现锁页内存（Pinning Memory）

Pinning Memory是空间换时间的做法，将指定的数据“锁”住，不会被系统移动（交换）到磁盘中的虚拟内存，因此可以加快数据的读取速率。

简单的可以理解为常用的衣服就“锁”在你的衣柜里，某些时候（如夏天），暂时不用的衣服——冬季大衣，则会移动到收纳柜里，以腾出空间放其它常用的衣服，等到冬天来临，需要用到大衣的时候，再从收纳柜里把大衣放到衣柜中。但是冬天拿大衣的时候就会慢一些，如果把它“锁”在你的衣柜，那么冬天获取它的时候自然快了，但占用了你的空间。这就是空间换时间的一个例子。

### DataLoader API

 - **dataset**：它是一个Dataset实例，要能实现从索引（indices/keys）到样本的映射。（即getitem函数）
 - **batch_size**：每个batch的样本量 shuffle：是否对打乱样本顺序。训练集通常要打乱它！验证集和测试集无所谓。
 -  **sampler**：设置采样策略。
 - **batch_sampler**：设置采样策略，batch_sampler与sampler二选一。 
 - **num_workers**：  设置多少个子进程进行数据加载（data loading） 
 - **collate_fn**：组装数据的规则， 决定如何将一批数据组装起来。 
 - **pin_memory**：是否使用锁页内存。
 -  **drop_last**：每个epoch是否放弃最后一批不足batchsize大小的数据，即无法被batchsize整除时，最后会有一小批数据，是否进行训练，如果数据量足够多，通常设置为True。这样使模型训练更为稳定，大家千万不要理解为某些数据被舍弃了，因为每个epoch，dataloader的采样都会重新shuffle，因此不会存在某些数据被真正的丢弃。
## transforms

数据增强（Data augmentation）已经成为深度学习时代的常规做法，数据增强目的是为了增加训练数据的丰富度，让模型接触多样性的数据以增加模型的泛化能力。

通常，数据增强可分为在线(online)与离线(offline)两种方式，离线方式指的是在训练开始之前将数据进行变换，变换后的图片保存到硬盘当中，在线方式则是在训练过程中，每一次加载训练数据时对数据进行变换，以实现让模型看到的图片都是增强之后的。实际上，这两种方法理论上是等价的，一般的框架都采用在线方式的数据增强，pytorch的transforms就是在线方式。

transforms是广泛使用的图像变换库，包含二十多种基础方法以及多种组合功能，通常可以用Compose把各方法串联在一起使用。大多数的transforms类都有对应的 functional transforms ，可供用户自定义调整。transforms提供的主要是PIL格式和Tensor的变换，并且对于图像的通道也做了规定，默认情况下一个batch的数据是(B, C, H, W) 形状的张量。
### transforms 的常用方法
```python
Resize：将图像缩放到指定大小。
RandomHorizontalFlip：随机水平翻转图像。
ToTensor：将 PIL.Image 或 numpy.ndarray 转换为 Tensor。
Normalize：对图像进行标准化处理。
RandomCrop：随机裁剪图像。
ColorJitter：随机改变图像的颜色。
Compose：组合多个转换。
```

PyTorch 不仅可设置对数据的操作，还可以对这些操作进行随机选择、组合，让数据增强更加灵活。具体有以下4个方法：

**Lambda**
Lambda 允许用户自定义转换函数。这对于实现一些特定的转换逻辑非常有用，例如自定义的数学运算或者特定的数据处理操作。

**RandomChoice**
RandomChoice 从给定的一组转换中随机选择一个应用。这对于数据增强非常有用，因为它可以增加训练数据的多样性。

**RandomOrder**
RandomOrder 会随机改变转换的执行顺序。这对于确保转换之间的顺序不固定，增加数据增强的随机性非常有用。

**RandomApply**
RandomApply 以一定的概率应用一组转换。这对于控制数据增强的强度非常有用，可以通过调整概率来控制转换是否被应用。

### 自动数据增强
自动数据增强的基本思想是将数据增强策略的选择视为一个优化问题，通过某种搜索算法来找到最优的数据增强策略。常见的搜索算法包括强化学习、遗传算法、贝叶斯优化等。

 - AutoAugment：由 Google 提出的一种方法，使用强化学习来搜索最佳的数据增强策略。它定义了一个搜索空间，其中每个策略由多个子策略组成，每个子策略包含两个操作及其对应的强度。
 - Fast AutoAugment：一种加速版本的 AutoAugment 方法，通过使用代理任务来减少搜索时间。
 - RandAugment：一种简化版的自动数据增强方法，它随机选择一系列操作并调整其强度，以减少搜索时间和计算资源的需求。
 - Policy Search：使用其他搜索算法如遗传算法、贝叶斯优化等来寻找最佳的数据增强策略。

参考：[https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-3/](https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-3/)
参考：[https://pytorch-cn.readthedocs.io/zh/latest/](https://pytorch-cn.readthedocs.io/zh/latest/)
参考：[https://datawhalechina.github.io/thorough-pytorch/](https://datawhalechina.github.io/thorough-pytorch/)
