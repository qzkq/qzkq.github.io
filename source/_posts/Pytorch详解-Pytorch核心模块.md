---
title: Pytorch详解-Pytorch核心模块
date: 2026-01-03 08:02:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Pytorch核心模块)
# 一、Pytorch模块结构
**通过`pip install`进行一键安装，工具库代码安装在了哪里？
使用的时候只知道`import *`，但具体引用的功能函数又是如何实现的？**

`import torch`这一行代码，按住Ctrl键，鼠标左键单击torch，就可以跳转到`__init__.py` 文件。右键即可在资源管理器里打开。
![在这里插入图片描述](/img/fad09b493a814ea99460dc6e0a97f5b4.png)
![Pytorch模块](/img/20b0f768b13d4e7dab995295a2cb428a.png)
## `_pycache_`
该文件夹存放python解释器生成的字节码，后缀通常为pyc/pyo。其目的是通过牺牲一定的存储空间来提高加载速度，对应的模块直接读取pyc文件，而不需再次将.py语言转换为字节码的过程，从此节省了时间。

从文件夹名称可知，它是一个缓存，如果需要，可以删掉它。

## `_C`
从文件夹名称就知道它和C语言有关，其实它是辅助C语言代码调用的一个模块，该文件夹里存放了一系列pyi文件，pyi文件是python用来校验数据类型的，如果调用数据类型不规范，会报错。

PyTorch的底层计算代码采用的是C++语言编写，并封装成库，供pytorch的python语言进行调用。一些pytorch函数无法跳转到具体实现，这是因为具体的实现通过C++语言，无法在Pycharm中跳转查看。

## include
C++代码在哪里？ 在torch/csrc文件夹下可以看到各个.h/.hpp文件，而在python库中，只包含头文件，这些头文件就在include文件夹下。

## lib
torch文件夹中最重要的一个模块，torch文件夹占1.06GB的空间，98%的内容都在lib中，占了0.98GB空间。

lib文件夹下包含大量的.lib .dll文件（分别是**静态链接库和动态链接库**）。底层库都会被各类顶层python api调用。

## autograd
实现了梯度的自动求导，极大地简化了深度学习研究者开发的工作量，开发人员只需编写前向传播代码，反向传播部分由autograd自动实现，再也不用手动去推导数学公式，然后编写代码了

## nn
搭建网络的网络层就在nn.modules里边。可以到Lib\site-packages\torch\nn\modules里面看看是否有你熟悉的网络层。

## optim
优化模块，深度学习的学习过程，就是不断的优化，而优化使用的方法函数，都暗藏在了optim文件夹中，进入该文件夹，可以看到熟悉的优化方法：“Adam”、“SGD”、“ASGD”等。以及非常重要的学习率调整模块，lr_scheduler.py。

## utils
utils是各种软件工程中常见的文件夹，其中包含了各类常用工具，其中比较关键的是data文件夹，tensorboard文件夹。

# 二、Lib\site-packages\torchvision

## datasets
官方为常用的数据集写的数据读取函数，例如常见的cifar, coco, mnist,svhn,voc都是有对应的函数支持，可以方便地使用轮子，同时也可以学习大牛们是如何写dataset的。

## models
里边存放了经典的、可复现的、有训练权重参数可下载的视觉模型，例如分类的alexnet、densenet、efficientnet、mobilenet-v1/2/3、resnet等，分割模型、检测模型、视频任务模型、量化模型。

## ops
视觉任务特殊的功能函数，例如检测中用到的 roi_align, roi_pool，boxes的生成，以及focal_loss实现，都在这里边有实现。

## transforms
数据增强库，transforms是pytorch自带的图像预处理、增强、转换工具，可以满足日常的需求。
# 三、核心数据结构——Tensor（张量）
在深度学习中张量表示的是一个多维数组，它是标量、向量、矩阵的拓展。标量是零维张量（数字），向量是一维张量，矩阵是二维张量，一个RGB图像的数组就是一个三维张量，第一维是图像的高，第二维是图像的宽，第三维是图像的颜色通道。
## 在深度学习中，时间序列数据为什么是三维张量？
在深度学习中，时间序列数据通常被表示为三维张量，这是因为它们需要符合特定的神经网络架构（如循环神经网络 RNNs 或长短时记忆网络 LSTM）的输入要求。这种表示方式有助于网络理解序列中的模式，并能够有效地处理序列数据。以下是三维张量的具体含义：

 1. 样本数量 (Samples): 第一个维度代表了数据集中有多少个独立的序列样本。例如，如果数据集包含多个股票的历史价格数据，则每个股票的历史价格序列就是一个样本。
 2. 时间步 (Time Steps): 第二个维度表示每个样本序列中的时间点数量。例如，如果每个股票的价格序列包含过去一年每天的收盘价格，则这个维度就是365天。 
 3. 特征数量 (Features): 第三个维度表示在每个时间点上有多少个特征。例如，除了收盘价格之外，还可能包括开盘价格、最高价格、最低价格等特征。

因此，一个典型的三维张量表示可以是 (samples, time_steps, features)。

在pytorch中，有两个张量的相关概念极其容易混淆，分别是torch.Tensor和torch.tensor。其实，通过命名规范，可知道torch.Tensor是Python的一个类, torch.tensor是Python的一个函数。通常我们调用torch.tensor进行创建张量，而不直接调用torch.Tensor类进行创建。

## 张量的作用
tensor之于pytorch等同于ndarray之于numpy，它是pytorch中最核心的数据结构，用于表达各类数据，如输入数据、模型的参数、模型的特征图、模型的输出等。这里边有一个很重要的数据，就是模型的参数。对于模型的参数，我们需要更新它们，而更新操作需要记录梯度，梯度的记录功能正是被张量所实现的（求梯度是autograd实现的）。
## 张量的结构
Tensor主要有以下八个主要属性，data，dtype，shape，device，grad，grad_fn，is_leaf，requires_grad。

 - data：多维数组，最核心的属性，其他属性都是为其服务的;
 - dtype：多维数组的数据类型;
 - shape：多维数组的形状;
 - device: tensor所在的设备，cpu或cuda;
 - grad: 对应于data的梯度，形状与data一致；
 - grad_fn: 记录创建该Tensor时用到的Function，该Function在反向传播计算中使用，因此是自动求导的关键；
 - is_leaf: 指示节点是否为叶子节点，为叶子结点时，反向传播结束，其梯度仍会保存，非叶子结点的梯度被释放，以节省内存;
 - requires_grad: 指示是否计算梯度；
# 四、张量的相关函数
## 张量的创建
### 直接创建
#### torch.tensor

```python
torch.tensor(data, dtype=None, device=None, requires_grad=False, pin_memory=False)
```

 - data(array_like) - tensor的初始数据，可以是list, tuple, numpy array,  scalar或其他类型。
 - dtype(torch.dtype, optional) - tensor的数据类型，如torch.uint8, torch.float, torch.long等
 - device (torch.device, optional) – 决定tensor位于cpu还是gpu。如果为None，将会采用默认值，默认值在torch.set_default_tensor_type()中设置，默认为cpu。
 - requires_grad (bool, optional) – 决定是否需要计算梯度。
 - pin_memory (bool, optional) – 是否将tensor存于锁页内存。这与内存的存储方式有关，通常为False。

#### torch.from_numpy

还有一种常用的通过numpy创建tensor方法是torch.from_numpy()。
**创建的tensor和原array共享同一块内存，即当改变array里的数值，tensor中的数值也会被改变。**

### 依数值创建
#### torch.zeros

```python
orch.zeros(*size, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：依给定的size创建一个全0的tensor，默认数据类型为torch.float32（也称为torch.float）。

主要参数：

 - layout(torch.layout, optional) - 参数表明张量在内存中采用何种布局方式。常用的有torch.strided, torch.sparse_coo等。
 - out(tensor, optional) - 输出的tensor，即该函数返回的tensor可以通过out进行赋值。

```python
import torch
o_t = torch.tensor([1])
t = torch.zeros((3, 3), out=o_t)
print(t, '\n', o_t)
print(id(t), id(o_t))
```

> tensor([[0, 0, 0],
> 
> ​ [0, 0, 0],
> 
> ​ [0, 0, 0]])
> 
> tensor([[0, 0, 0],
> 
> ​ [0, 0, 0],
> 
> ​ [0, 0, 0]])
> 
> 4925603056 4925603056


通过torch.zeros创建的张量不仅赋给了t，同时赋给了o_t，并且这两个张量是共享同一块内存，只是变量名不同。

#### torch.strided

一种表示稠密张量的方式，其中数据连续存储在内存中，并通过步长（strides）来访问多维数组中的元素。大多数常见的 PyTorch 张量都是 strided 张量。

#### torch.sparse_coo

这种格式用于表示稀疏张量，特别是坐标列表（COOrdinate List）格式。在这种格式下，只有非零值及其对应的索引被存储。
对于大部分非零元素很少的大矩阵来说，使用 torch.sparse_coo 格式可以节省大量的内存空间。
稀疏张量的操作通常比稠密张量的操作更复杂，因为需要处理不连续的数据存储。

#### torch.zeros_like

```python
torch.zeros_like(input, dtype=None, layout=None, device=None, requires_grad=False)
```

功能：依input的size创建全0的tensor。

主要参数：

 - input(Tensor) - 创建的tensor与intput具有相同的形状。

```python
torch.ones(*size, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：依给定的size创建一个全1的tensor。

```python
torch.ones_like(input, dtype=None, layout=None, device=None, requires_grad=False)
```

功能：依input的size创建全1的tensor。

```python
torch.full(size, fill_value, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：依给定的size创建一个值全为fill_value的tensor。

主要参数:

 - siz (int...) - tensor的形状。
 - fill_value - 所创建tensor的值
 - out(tensor, optional) - 输出的tensor，即该函数返回的tensor可以通过out进行赋值。

```python
torch.full_like(input, fill_value, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

torch.full_like之于torch.full等同于torch.zeros_like之于torch.zeros

```python
torch.arange(start=0, end, step=1, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：创建等差的1维张量，长度为 (end-start)/step，需要注意数值区间为[start, end)。

主要参数：

 - start (Number) – 数列起始值，默认值为0。the starting value for the set of points. Default: 0.
 - end (Number) – 数列的结束值。
 - step (Number) – 数列的等差值，默认值为1。
 - out (Tensor, optional) – 输出的tensor，即该函数返回的tensor可以通过out进行赋值。

```python
torch.linspace(start, end, steps=100, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：创建均分的1维张量，长度为steps，区间为[start, end]。

主要参数：

 - start (float) – 数列起始值。
 - end (float) – 数列结束值。
 - steps (int) – 数列长度。

```python
torch.logspace(start, end, steps=100, base=10.0, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：创建对数均分的1维张量，长度为steps, 底为base。

主要参数：

 - start (float) – 确定数列起始值为base^start
 - end (float) – 确定数列结束值为base^end
 - steps (int) – 数列长度。
 - base (float) - 对数函数的底，默认值为10

```python
torch.empty(*size, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False, pin_memory=False)
```

功能：依size创建“空”张量，这里的“空”指的是不会进行初始化赋值操作。

主要参数：

 - size (int...) - 张量维度
 - pin_memory (bool, optional) - pinned memory 又称page locked memory，即锁页内存，该参数用来指示是否将tensor存于锁页内存，通常为False，若内存足够大，建议设置为True，这样在转到GPU时会快一些。

```python
torch.empty_like(input, dtype=None, layout=None, device=None, requires_grad=False)
```

功能：torch.empty_like之于torch.empty等同于torch.zeros_like之于torch.zeros，因此不再赘述。

```python
torch.empty_strided(size, stride, dtype=None, layout=None, device=None, requires_grad=False, pin_memory=False)
```

功能：依size创建“空”张量，这里的“空”指的是不会进行初始化赋值操作。

主要参数：

 - stride (tuple of python:ints) - 张量存储在内存中的步长，是设置在内存中的存储方式。
 - size (int...) - 张量维度
 - pin_memory (bool, optional) - 是否存于锁页内存。

### 依概率分布创建

```python
torch.normal(mean, std, out=None)
```

功能：为每一个元素以给定的mean和std用高斯分布生成随机数

主要参数：

 - mean (Tensor or Float) - 高斯分布的均值，
 - std (Tensor or Float) - 高斯分布的标准差

```python
mean为张量，std为张量，torch.normal(mean, std, out=None)，每个元素从不同的高斯分布采样，分布的均值和标准差由mean和std对应位置元素的值确定；

mean为张量，std为标量，torch.normal(mean, std=1.0, out=None)，每个元素采用相同的标准差，不同的均值；

mean为标量，std为张量，torch.normal(mean=0.0, std, out=None)， 每个元素采用相同均值，不同标准差；

mean为标量，std为标量，torch.normal(mean, std, size, *, out=None) ，从一个高斯分布中生成大小为size的张量；
```

```python
torch.rand(*size, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：在区间[0, 1)上，生成均匀分布。

主要参数：

 - size (int...) - 创建的张量的形状

```python
torch.rand_like(input, dtype=None, layout=None, device=None, requires_grad=False)
```

torch.rand_like之于torch.rand等同于torch.zeros_like之于torch.zeros。

```python
torch.randint(low=0, high, size, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：在区间[low, high)上，生成整数的均匀分布。

主要参数：

 - low (int, optional) - 下限。
 - high (int) – 上限，主要是开区间。
 - size (tuple) – 张量的形状。

```python
torch.randint_like(input, low=0, high, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：torch.randint_like之于torch.randint等同于torch.zeros_like之于torch.zeros。

```python
torch.randn(*size, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False)
```

功能：生成形状为size的标准正态分布张量。

主要参数：

 - size (int...) - 张量的形状

```python
torch.randn_like(input, dtype=None, layout=None, device=None, requires_grad=False)
```

功能：torch.rafndn_like之于torch_randn等同于torch.zeros_like之于torch.zeros。

```python
torch.randperm(n, out=None, dtype=torch.int64, layout=torch.strided, device=None, requires_grad=False)
```

功能：生成从0到n-1的随机排列。perm == permutation

```python
torch.bernoulli(input, *, generator=None, out=None)
```

功能：以input的值为概率，生成伯努力分布（0-1分布，两点分布）。

主要参数：

 - input (Tensor) - 分布的概率值，该张量中的每个值的值域为[0-1]

## 张量的操作
Tensor与numpy的数据结构很类似，不仅数据结构类似，操作也是类似的。

```python
cat	将多个张量拼接在一起，例如多个特征图的融合可用。
concat	同cat, 是cat()的别名。
conj	返回共轭复数。
chunk	将tensor在某个维度上分成n份。
dsplit	类似numpy.dsplit().， 将张量按索引或指定的份数进行切分。
column_stack	水平堆叠张量。即第二个维度上增加，等同于torch.hstack。
dstack	沿第三个轴进行逐像素（depthwise）拼接。
gather	高级索引方法，目标检测中常用于索引bbox。在指定的轴上，根据给定的index进行索引。
hsplit	类似numpy.hsplit()，将张量按列进行切分。若传入整数，则按等分划分。若传入list，则按list中元素进行索引。例如：[2, 3] and dim=0 would result in the tensors input[:2], input[2:3], and input[3:].
hstack	水平堆叠张量。即第二个维度上增加，等同于torch.column_stack。
index_select	在指定的维度上，按索引进行选择数据，然后拼接成新张量。可知道，新张量的指定维度上长度是index的长度。
masked_select	根据mask（0/1, False/True 形式的mask）索引数据，返回1-D张量。
movedim	移动轴。如0，1轴交换：torch.movedim(t, 1, 0) .
moveaxis	同movedim。Alias for torch.movedim().（这里发现pytorch很多地方会将dim和axis混用，概念都是一样的。）
narrow	变窄的张量？从功能看还是索引。在指定轴上，设置起始和长度进行索引。例如：torch.narrow(x, 0, 0, 2)， 从第0个轴上的第0元素开始，索引2个元素。x[0:0+2, ...]
nonzero	返回非零元素的index。torch.nonzero(torch.tensor([1, 1, 1, 0, 1])) 返回tensor([[ 0], [ 1], [ 2], [ 4]])。建议看example，一看就明白，尤其是对角线矩阵的那个例子，太清晰了。
permute	交换轴。
reshape	变换形状。
row_stack	按行堆叠张量。即第一个维度上增加，等同于torch.vstack。Alias of torch.vstack().
scatter	scatter_(dim, index, src, reduce=None) → Tensor。将src中数据根据index中的索引按照dim的方向填进input中。这是一个十分难理解的函数，其中index是告诉你哪些位置需要变，src是告诉你要变的值是什么。这个就必须配合例子讲解，请跳转到本节底部进行学习。
scatter_add	同scatter一样，对input进行元素修改，这里是 +=， 而scatter是直接替换。
split	按给定的大小切分出多个张量。例如：torch.split(a, [1,4])； torch.split(a, 2)
squeeze	移除张量为1的轴。如t.shape=[1, 3, 224, 224]. t.squeeze().shape -> [3, 224, 224]
stack	在新的轴上拼接张量。与hstack\vstack不同，它是新增一个轴。默认从第0个轴插入新轴。
swapaxes	Alias for torch.transpose().交换轴。
swapdims	Alias for torch.transpose().交换轴。
t	转置。
take	取张量中的某些元素，返回的是1D张量。torch.take(src, torch.tensor([0, 2, 5]))表示取第0,2,5个元素。
take_along_dim	取张量中的某些元素，返回的张量与index维度保持一致。可搭配torch.argmax(t)和torch.argsort使用，用于对最大概率所在位置取值，或进行排序，详见官方文档的example。
tensor_split	切分张量，核心看indices_or_sections变量如何设置。
tile	将张量重复X遍，X遍表示可按多个维度进行重复。例如：torch.tile(y, (2, 2))
transpose	交换轴。
unbind	移除张量的某个轴，并返回一串张量。如[[1], [2], [3]] --> [1], [2], [3] 。把行这个轴拆了。
unsqueeze	增加一个轴，常用于匹配数据维度。
vsplit	垂直切分。
vstack	垂直堆叠。
where	根据一个是非条件，选择x的元素还是y的元素，拼接成新张量。看案例可瞬间明白。
```

### scater_

在 PyTorch 中，scatter_ 是一个用于在张量的特定维度上根据索引进行散列操作的方法。它允许您将指定位置的值更新为新的值。scatter_ 方法是在原地（in-place）操作的，意味着它会直接修改原始张量。

```python
t.scatter_(dim, index, src)
```

参数说明:

 - dim: 指定要在哪个维度上执行散列操作。
 - index: 包含目标位置索引的整数张量。
 - src: 包含新值的张量或单个数值。

**示例**：假设我们有一个 2D 张量 t，并希望根据索引更新某些值：

```python
import torch

# 创建一个 3x3 的张量
t = torch.zeros(3, 3)
print("Original tensor:")
print(t)

# 更新第 1 行的值
index = torch.tensor([[0, 1, 2]])
src = torch.tensor([[-1.0, -2.0, -3.0]])
t.scatter_(0, index, src)
print("\nAfter scatter operation:")
print(t)
```
输出结果将是：

```python
Original tensor:
tensor([[0., 0., 0.],
        [0., 0., 0.],
        [0., 0., 0.]])

After scatter operation:
tensor([[-1., -2., -3.],
        [ 0.,  0.,  0.],
        [ 0.,  0.,  0.]])
```
在这个例子中，scatter_ 将第 1 行的值更新为 -1.0, -2.0, -3.0。
**注意事项**
scatter_ 是 in-place 操作，这意味着它会直接修改原始张量而不是返回一个新的张量。确保索引张量和源张量的形状兼容，以避免错误。
**总结**
scatter_ 是一个非常有用的工具，特别是在需要根据索引更新张量中的值时。它可以用于实现各种数据处理任务，如掩码操作、条件赋值等。

## 张量的随机种子（CPU）
随机种子（random seed）主要用于实验的复现。

```python
seed	获取一个随机的随机种子。Returns a 64 bit number used to seed the RNG.
manual_seed	手动设置随机种子，建议设置为42，这是近期一个玄学研究。说42有效的提高模型精度。当然大家可以设置为你喜欢的，只要保持一致即可。
initial_seed	返回初始种子。
get_rng_state	获取随机数生成器状态。Returns the random number generator state as a torch.ByteTensor.
```

## 张量的数学操作
张量还提供大量数学操作，

```python
Pointwise Ops： 逐元素的操作，如abs, cos, sin, floor, floor_divide, pow等
Reduction Ops: 减少元素的操作，如argmax, argmin, all, any, mean, norm, var等
Comparison Ops：对比操作， 如ge, gt, le, lt, eq, argsort, isnan, topk,
Spectral Ops: 谱操作，如短时傅里叶变换等各类信号处理的函数。
Other Operations：其它， clone， diag，flip等
BLAS and LAPACK Operations：BLAS（Basic Linear Algebra Subprograms）基础线性代数）操作。如, addmm, dot, inner, svd等。
```
# 五、计算图（Computational Graphs）
计算图（Computational Graphs）是一种描述运算的“语言”，它由节点(Node)和边(Edge)构成。

 - 节点（Nodes）: 节点代表了计算图中的变量或操作。每个节点可以是输入变量、中间变量或输出变量。
 - 边（Edges）: 边连接节点，表示数据流的方向和依赖关系。从输入到输出，数据沿着边流动。

**非叶子结点在梯度反向传播结束后释放**

只有叶子节点的梯度得到保留，中间变量的梯度默认不保留；在pytorch中，非叶子结点的梯度在反向传播结束之后就会被释放掉，如果需要保留的话可以对该结点设置retain_grad()

**grad_fn是用来记录创建张量时所用到的运算，在链式求导法则中会使用到。**
## 静态图与动态图
### 静态图（Static Graph）
在静态图中，计算图在执行之前就已经定义好了，即在数据输入之前就已经构建完成。计算图一旦构建，其结构在整个程序运行期间不会改变。
**构建与执行:**

 - 首先定义计算图中的所有操作和变量。
 - 然后将数据输入到已经构建好的计算图中进行计算。

**优点:**

 - 由于计算图在运行前已经构建完成，因此可以进行更多的优化，如静态分析、图级优化等。
 - 更容易并行化和分布式计算。

**缺点:**

 - 构建计算图需要额外的步骤，增加了编程的复杂度。
 - 不适合处理动态数据流或控制流，如循环次数未知的情况。

### 动态图（Dynamic Graph）
在动态图中，计算图是在运行时根据数据的流动动态地构建和执行的。计算图的结构可以根据数据的变化而变化。
**构建与执行:**

 - 在执行计算时，每一步的操作都会立即被执行。
 - 每次执行可能会构建不同的计算图，这取决于输入数据和控制流。

**优点:**

 - 更加灵活，可以更容易地处理控制流和动态数据结构。
 - 编程更加直观，类似于普通的函数式编程。

**缺点:**

 - 由于计算图是动态构建的，可能难以进行优化和并行化。
 - 可能会导致性能上的开销，尤其是对于复杂的控制流。


# 六、Autograd - 自动微分
**Autograd 的工作原理**

Autograd 通过构建计算图来跟踪张量操作的依赖关系，并能够高效地计算梯度。具体来说：

 - 构建计算图:

当对带有 .requires_grad=True 的张量执行操作时，PyTorch 会在后台自动构建计算图。
计算图记录了从输入到输出的所有操作。

 - 前向传播:

执行计算图中的操作来获得输出。
在这个过程中，计算图会记录每个操作的元数据，以便后续的反向传播。

 - 反向传播:

通过调用 .backward() 方法来计算梯度。
利用链式法则，从输出开始逐层向前计算每个张量的梯度。

 - 梯度更新:

利用计算出的梯度来更新模型参数。
通常通过优化器（如 SGD、Adam 等）来实现参数更新。

**关键概念**

 - 可求导张量:

通过设置 requires_grad=True 来创建可求导的张量。
这些张量是构建计算图的基础。

 - 计算图:

计算图是一个有向无环图（DAG），记录了张量操作的依赖关系。
计算图用于追踪前向传播过程中的所有操作，并支持高效的反向传播。

 - 反向传播:

通过调用 .backward() 方法来触发反向传播过程。
反向传播计算每个张量的梯度，并将其存储在 .grad 属性中。

 - 优化器:

优化器（如 torch.optim.SGD、torch.optim.Adam 等）负责更新模型参数。
优化器使用计算出的梯度来更新参数，以最小化损失函数。

```python
自动求导机制通过有向无环图（directed acyclic graph ，DAG）实现
在DAG中，记录数据（对应tensor.data）以及操作（对应tensor.grad_fn）
操作在pytorch中统称为Function，如加法、减法、乘法、ReLU、conv、Pooling等，统统是Function
```
## autograd 的使用
Autograd 是 PyTorch 中的一个自动微分模块，它提供了自动计算张量操作梯度的能力。Autograd 是 PyTorch 的核心特性之一，使得用户能够轻松地构建和训练深度学习模型，而无需手动编写梯度计算代码。
### torch.autograd.backward
torch.autograd.backward 是 PyTorch 中用于触发反向传播过程的函数，它用于计算损失函数关于模型参数的梯度。

```python
torch.autograd.backward(tensors, grad_tensors=None, retain_graph=None, create_graph=False, grad_variables=None, inputs=None)
```

 - tensors: 一个包含张量的列表或元组，这些张量需要计算梯度。
 - grad_tensors: 一个与 tensors 相同长度的列表或元组，包含每个张量的梯度张量。如果未提供，则默认为每个张量的梯度为1。
 - retain_graph: 一个布尔值，表示是否保留计算图以供后续的反向传播。默认为 False。
 - create_graph: 一个布尔值，表示是否为梯度计算构建计算图。默认为 False。如果为 True，则可以进一步计算梯度的梯度。

**注意事项**

 - 梯度清零:

在每次反向传播之前，需要清零梯度，否则梯度会被累加。可以使用 optimizer.zero_grad() 或者 x.grad.data.zero_() 来实现。

 - 保留计算图:

如果需要多次反向传播，可以设置 retain_graph=True 来保留计算图。这样可以在不重建计算图的情况下进行多次反向传播。

 - 梯度累积:

默认情况下，.backward() 会将梯度累加到现有的梯度中。如果不需要累加梯度，可以在调用 .backward() 之前清零梯度。

 - 梯度张量:

可以通过提供 grad_tensors 参数来指定每个张量的梯度张量。这对于某些特殊情况下的梯度计算很有用。

 - 梯度计算图:

如果需要计算梯度的梯度，可以设置 create_graph=True。这在某些高级用例中很有用，例如高阶微分或梯度惩罚。
### torch.autograd.grad

```python
torch.autograd.grad(outputs, inputs, grad_outputs=None, retain_graph=None, create_graph=False, only_inputs=True, allow_unused=False)
```

**功能：计算outputs对inputs的导数**

**参数说明:**

 - outputs: 一个包含输出张量的列表或元组，这些张量的梯度需要被计算。
 - inputs: 一个包含输入张量的列表或元组，这些张量的梯度需要被计算。
 - grad_outputs: 一个与 outputs 相同长度的列表或元组，包含每个输出张量的梯度张量。如果未提供，则默认为每个输出张量的梯度为 1。
 - retain_graph: 一个布尔值，表示是否保留计算图以供后续的反向传播。默认为 False。
 - create_graph: 一个布尔值，表示是否为梯度计算构建计算图。默认为 False。如果为 True，则可以进一步计算梯度的梯度。
 - only_inputs: 一个布尔值，表示是否只返回 inputs 中张量的梯度。默认为 True。
 - allow_unused: 一个布尔值，表示是否允许某些输入张量没有被使用。如果为 True，则未使用的输入张量的梯度将被设置为 None。
### torch.autograd.Function
torch.autograd.Function 是 PyTorch 中的一个基类，用于自定义新的张量操作及其对应的反向传播。通过继承 torch.autograd.Function 并重写前向传播 (forward) 和反向传播 (backward) 方法，可以实现自定义的操作，并使其能够与 PyTorch 的自动微分机制无缝集成。
## autograd相关的知识点
### 梯度不会自动清零
在 PyTorch 中，梯度不会自动清零是一个重要的概念，这意味着在每次反向传播之后，梯度会被累加到现有的梯度上。这一设计有其特定的目的和优点，但也需要开发者注意在训练模型时正确地管理梯度。
#### 梯度累加的原因
**多任务学习:**
在多任务学习中，一个模型可能需要同时优化多个目标函数。梯度累加可以方便地实现这一点，因为每次反向传播之后，梯度会被累加到现有的梯度上，从而可以同时考虑多个任务的梯度信息。
**灵活性:**
开发者可以自由选择何时清零梯度，比如在训练过程中每 N 个 batch 清零一次梯度，这样可以实现梯度累积的效果，有助于优化器更好地调整学习率。
**内存效率:**
由于梯度是累加的，因此可以减少内存的使用，尤其是在多任务学习场景下，不需要为每个任务单独分配内存来存储梯度。
#### 如何管理梯度
**梯度清零:**
在每次反向传播之前，通常需要清零梯度，以避免梯度累加导致的问题。可以使用 optimizer.zero_grad() 来清零优化器管理的所有参数的梯度。
**反向传播:**
调用 .backward() 方法来计算梯度。
如果需要多次反向传播，可以设置 retain_graph=True 来保留计算图。
**参数更新:**
使用优化器（如 torch.optim.SGD, torch.optim.Adam 等）来更新模型参数。
通过调用 optimizer.step() 方法来应用梯度更新。
### 依赖于叶子结点的结点，requires_grad默认为True
**注意事项**
**梯度计算:**
只有 requires_grad=True 的张量才会被计算梯度。
如果一个张量的 requires_grad 属性为 False，即使它依赖于 requires_grad=True 的叶子节点，该张量也不会被计算梯度。
**梯度清零:**
在每次反向传播之前，需要清零梯度，否则梯度会被累加。可以使用 optimizer.zero_grad() 或者 x.grad.data.zero_() 来实现。
**requires_grad 的修改:**
可以通过 .requires_grad_(new_value) 方法来修改张量的 requires_grad 属性。
如果一个张量的 requires_grad 属性被修改为 False，那么依赖于该张量的其他张量的 requires_grad 属性也可能受到影响。
### 叶子张量不可以执行in-place操作
在 PyTorch 中，叶子张量（leaf tensor）是指那些没有父节点的张量，通常是用户直接创建的张量。叶子张量的 requires_grad 属性决定了是否需要计算该张量的梯度。对于叶子张量，执行 in-place 操作可能会导致一些问题，主要是因为 in-place 操作会破坏计算图的完整性，进而影响梯度的计算。
#### 什么是 in-place 操作
In-place 操作是指直接修改现有张量内容的操作，而不是创建一个新的张量。这类操作通常以 _ 结尾，例如 add_()、mul_() 等。
#### 为什么叶子张量不能执行 in-place 操作
**计算图的完整性:**
PyTorch 的自动微分机制依赖于构建完整的计算图来追踪张量操作的历史。
当对叶子张量执行 in-place 操作时，PyTorch 无法追踪这一操作，因为它直接修改了原始张量而没有创建新的张量。
这会导致计算图丢失信息，从而在反向传播时无法正确计算梯度。
**梯度计算:**
如果叶子张量执行了 in-place 操作，那么依赖于该叶子张量的计算图就无法正确反映实际的操作历史。
这意味着在反向传播时，PyTorch 无法正确地计算叶子张量的梯度，从而可能导致梯度计算错误或不完整。
**解决方案**
**使用非 in-place 操作:**
为了避免问题，可以使用非 in-place 操作，例如 add() 而不是 add_()。
这样会创建一个新的张量，而原始张量保持不变，从而保证计算图的完整性。
**使用 torch.no_grad() 上下文管理器:**
如果确实需要执行 in-place 操作，并且确定不需要计算梯度，可以使用 torch.no_grad() 上下文管理器。
这样可以暂时禁用梯度计算，从而允许执行 in-place 操作。
**转换为非叶子张量:**
如果需要执行 in-place 操作，并且仍然需要计算梯度，可以先将叶子张量转换为非叶子张量。
通过创建一个新的张量，例如 x = x.clone().detach().requires_grad_(True)，可以得到一个具有相同数据的新张量，但不再是叶子张量。
### detach 的作用
在 PyTorch 中，detach() 函数是一个非常有用的工具，用于从计算图中分离出一个张量，以便它可以被用作进一步计算的一部分而不参与梯度的计算。
从计算图中剥离出“数据”，并以一个新张量的形式返回，并且新张量与旧张量共享数据，简单的可理解为做了一个别名。

**分离张量:**
detach() 方法会创建一个新的张量，该张量与原始张量共享相同的底层数据，但不再追踪梯度信息。
分离后的张量永远不需要计算其梯度，即它的 requires_grad 属性被设置为 False。
**避免梯度传播:**
通过分离张量，你可以阻止梯度从分离的张量向后传播。这在一些情况下很有用，比如在训练过程中冻结某些参数，只更新部分参数。
**提取值:**
分离后的张量可以用于提取其数值，而不关心梯度信息。
**使用场景**
**冻结层:**
当你想冻结模型的某些层，只更新模型的其他部分时，可以使用 detach() 来阻止梯度传播到这些层。
**梯度裁剪:**
在训练循环中，有时需要对梯度进行裁剪，以防止梯度爆炸或消失。在这种情况下，可以使用 detach() 来创建一个新的张量，然后对这个张量进行裁剪。
**评估模式:**
在模型的评估阶段，通常不需要计算梯度。此时可以使用 detach() 来提高性能，因为不需要维护计算图。
**梯度计算控制:**
在某些情况下，你可能希望控制哪些张量的梯度被计算。例如，在强化学习中，你可能需要根据策略网络的输出来计算奖励，但不希望这个计算影响到策略网络的梯度。
### with torch.no_grad()的作用
with torch.no_grad() 是 PyTorch 中的一个上下文管理器，用于临时禁止计算图中的梯度计算。在该上下文管理器内部，所有张量的 requires_grad 属性都会被设置为 False，这意味着任何在此上下文中创建或操作的张量都不会追踪梯度信息。
**节省内存:**
在 with torch.no_grad() 内部，由于不需要计算梯度，所以可以节省大量的内存。这是因为计算图中的历史信息（用于梯度计算）不会被保存。
**提高性能:**
禁止梯度计算可以提高计算速度，尤其是在不需要梯度的场景下，比如模型推理阶段。
**避免梯度计算:**
在某些情况下，你可能希望避免对某些张量进行梯度计算。例如，在模型的评估阶段或者在生成模型的输出时。
**使用场景**
**模型评估:**
在模型评估阶段，通常不需要计算梯度。使用 with torch.no_grad() 可以提高评估的速度并减少内存使用。
**生成模型输出:**
当使用生成模型生成数据时，通常不需要计算梯度。使用 with torch.no_grad() 可以提高生成速度并减少内存使用。
**模型推理:**
在部署模型进行推理时，通常不需要梯度计算。使用 with torch.no_grad() 可以提高推理速度并减少内存使用。
**冻结层:**
当你想冻结模型的某些层，只更新模型的其他部分时，可以使用 with torch.no_grad() 来阻止梯度传播到这些层。
**避免梯度裁剪:**
在训练循环中，有时需要对梯度进行裁剪，以防止梯度爆炸或消失。在这种情况下，可以使用 with torch.no_grad() 来创建一个新的张量，然后对这个张量进行裁剪。

# 示例

```python
def predict(test_loader, model, device):
    model.eval() # 设置成eval模式.
    preds = []
    for x in tqdm(test_loader):
        x = x.to(device)                        
        with torch.no_grad():
            pred = model(x)         
            preds.append(pred.detach().cpu())   
    preds = torch.cat(preds, dim=0).numpy()  
    return preds

def trainer(train_loader, valid_loader, model, config, device):

    criterion = nn.MSELoss(reduction='mean') # 损失函数的定义

    # 定义优化器
    # TODO: 可以查看学习更多的优化器 https://pytorch.org/docs/stable/optim.html 
    # TODO: L2 正则( 可以使用optimizer(weight decay...) )或者 自己实现L2正则.
    optimizer = torch.optim.SGD(model.parameters(), lr=config['learning_rate'], momentum=0.9) 
    
    # tensorboard 的记录器
    writer = SummaryWriter()

    if not os.path.isdir('./models'):
        # 创建文件夹-用于存储模型
        os.mkdir('./models')

    n_epochs, best_loss, step, early_stop_count = config['n_epochs'], math.inf, 0, 0

    for epoch in range(n_epochs):
        model.train() # 训练模式
        loss_record = []

        # tqdm可以帮助我们显示训练的进度  
        train_pbar = tqdm(train_loader, position=0, leave=True)
        # 设置进度条的左边 ： 显示第几个Epoch了
        train_pbar.set_description(f'Epoch [{epoch+1}/{n_epochs}]')
        for x, y in train_pbar:
            optimizer.zero_grad()               # 将梯度置0.
            x, y = x.to(device), y.to(device)   # 将数据一到相应的存储位置(CPU/GPU)
            pred = model(x)             
            loss = criterion(pred, y)
            loss.backward()                     # 反向传播 计算梯度.
            optimizer.step()                    # 更新网络参数
            step += 1
            loss_record.append(loss.detach().item())
            
            # 训练完一个batch的数据，将loss 显示在进度条的右边
            train_pbar.set_postfix({'loss': loss.detach().item()})

        mean_train_loss = sum(loss_record)/len(loss_record)
        # 每个epoch,在tensorboard 中记录训练的损失（后面可以展示出来）
        writer.add_scalar('Loss/train', mean_train_loss, step)

        model.eval() # 将模型设置成 evaluation 模式.
        loss_record = []
        for x, y in valid_loader:
            x, y = x.to(device), y.to(device)
            with torch.no_grad():
                pred = model(x)
                loss = criterion(pred, y)

            loss_record.append(loss.item())
            
        mean_valid_loss = sum(loss_record)/len(loss_record)
        print(f'Epoch [{epoch+1}/{n_epochs}]: Train loss: {mean_train_loss:.4f}, Valid loss: {mean_valid_loss:.4f}')
        # 每个epoch,在tensorboard 中记录验证的损失（后面可以展示出来）
        writer.add_scalar('Loss/valid', mean_valid_loss, step)

        if mean_valid_loss < best_loss:
            best_loss = mean_valid_loss
            torch.save(model.state_dict(), config['save_path']) # 模型保存
            print('Saving model with loss {:.3f}...'.format(best_loss))
            early_stop_count = 0
        else: 
            early_stop_count += 1

        if early_stop_count >= config['early_stop']:
            print('\nModel is not improving, so we halt the training session.')
            return

device = 'cuda' if torch.cuda.is_available() else 'cpu'
config = {
    'seed': 5201314,      # 随机种子，可以自己填写. :)
    'select_all': True,   # 是否选择全部的特征
    'valid_ratio': 0.2,   # 验证集大小(validation_size) = 训练集大小(train_size) * 验证数据占比(valid_ratio)
    'n_epochs': 2000,     # 数据遍历训练次数           
    'batch_size': 256, 
    'learning_rate': 1e-5,              
    'early_stop': 400,    # 如果early_stop轮损失没有下降就停止训练.     
    'save_path': './models/model.ckpt'  # 模型存储的位置
}

# 使用Pytorch中Dataloader类按照Batch将数据集加载
train_loader = DataLoader(train_dataset, batch_size=config['batch_size'], shuffle=True, pin_memory=True)
valid_loader = DataLoader(valid_dataset, batch_size=config['batch_size'], shuffle=True, pin_memory=True)
test_loader = DataLoader(test_dataset, batch_size=config['batch_size'], shuffle=False, pin_memory=True)

model = My_Model(input_dim=x_train.shape[1]).to(device) # 将模型和训练数据放在相同的存储位置(CPU/GPU)

trainer(train_loader, valid_loader, model, config, device)

model = My_Model(input_dim=x_train.shape[1]).to(device)

model.load_state_dict(torch.load(config['save_path']))

preds = predict(test_loader, model, device) 
```

参考：[https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-2/](https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-2/)
参考：[https://pytorch-cn.readthedocs.io/zh/latest/](https://pytorch-cn.readthedocs.io/zh/latest/)
参考：[https://datawhalechina.github.io/thorough-pytorch/](https://datawhalechina.github.io/thorough-pytorch/)
