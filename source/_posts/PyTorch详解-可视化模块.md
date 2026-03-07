---
title: PyTorch详解-可视化模块
date: 2026-01-03 08:04:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](PyTorch详解-可视化模块)

## Tensorboard 基础与使用
TensorBoard 是一个非常流行的工具，用于可视化 TensorFlow 和 PyTorch 等深度学习框架的训练过程。它可以用来监控训练过程中的各种指标，如损失函数、准确率等，并且可以直观地展示模型结构、权重分布等信息。
### 启动 TensorBoard
启动 TensorBoard 需要在命令行中指定日志文件所在的目录。

```python
tensorboard --logdir=./logs
```

如果系统找不到 tensorboard 命令，可能需要使用绝对路径调用它：

```python
python -m tensorboard.main --logdir=./logs
```
### 访问 TensorBoard
启动后，TensorBoard 会默认在本地服务器的端口 6006 上运行。可以通过浏览器访问 [http://localhost:6006](http://localhost:6006) 来查看 TensorBoard 的界面。
### 使用 TensorBoard

```python
TensorBoard 提供了多个不同的面板来展示不同的信息：
Scalars：显示标量值的变化，如损失函数或准确率。
Graphs：显示计算图，包括操作节点和张量流。
Images：显示图像数据，可用于查看输入图像或中间层的激活。
Histograms：显示权重、偏置或其他变量的直方图。
Distributions and Profiles：显示分布信息和性能分析。
Embeddings：显示高维数据的嵌入表示。
Pruning：显示模型剪枝的信息。
Hyperparameter Tuning：显示超参数搜索的结果。
Projector：用于探索高维数据的可视化工具。
```

### SummaryWriter类介绍
SummaryWriter 类位于 torch.utils.tensorboard 模块中，用于记录训练过程中的各种数据，如标量、图像、直方图等，并将这些数据写入磁盘，以便 TensorBoard 可视化工具加载和显示。

```python
CLASS torch.utils.tensorboard.writer.SummaryWriter(log_dir=None, comment='', purge_step=None, max_queue=10, flush_secs=120, filename_suffix='')
```
#### 参数说明
**log_dir (str, optional)：**

 - 日志文件的保存位置，默认为 'runs/experiment_name'。
 - 如果省略，则默认在当前工作目录下的 runs 文件夹中创建一个新的子文件夹。
 - 示例：'runs/my_experiment'。

**comment (str, optional)：**

 - 添加到日志文件夹名称后的评论字符串。
 - 默认为空字符串 ''。
**purge_step (int, optional)：**

 - 删除旧的事件文件的步骤数。
 - 如果为 None，则不会删除任何文件。
 - 如果为整数，则在达到此步骤数后，将删除旧的事件文件。
**max_queue (int, optional)：**

 - 待写入的日志事件的最大队列大小，默认为 10。
 - 较大的队列可以减少写入磁盘的频率，从而提高性能。
**flush_secs (int, optional)：**

 - 写入事件到磁盘的间隔时间（秒），默认为 120 秒。
 - 较短的时间间隔可以更快地看到更新，但可能会增加 I/O 开销。
**filename_suffix (str, optional)：**

 - 附加到事件文件名末尾的后缀，默认为空字符串 ''。
 - 可以用于区分不同的日志文件。


#### 常用方法
**add_scalar(tag, scalar_value, global_step=None, walltime=None)**

 - tag：字符串形式的标签，用于区分不同类型的标量数据。 
 - scalar_value：要记录的标量值。
 - global_step：全局步骤数，用于绘制随时间变化的趋势图。 
 - walltime：可选的时间戳，用于精确控制数据点的时间。
 - 功能：添加标量；tag的设置可以有个技巧是在同一栏下绘制多个图，如'Loss/train'， 'Loss/Valid'， 这就类似于matplotlib的subplot(121), subplot(122)

**add_scalars(main_tag, tag_scalar_dict, global_step=None, walltime=None)**

 - 功能：在一个坐标轴中绘制多条曲线。常用于曲线对比。

 **add_image(tag, img_tensor, global_step=None, dataformats='CHW')**
   
 - tag：字符串形式的标签，用于区分不同类型的图像数据。 
 - img_tensor：图像数据，通常为 torch.Tensor 类型。
 -  global_step：全局步骤数。 
 - dataformats：图像数据的维度顺序，默认为 'CHW'（通道、高度、宽度）。
 - 功能：绘制图像。

**add_images(tag, img_tensor, global_step=None, walltime=None, dataformats='NCHW')**

 - 功能：绘制图像序列，常用于数据清洗，卷积核，特征图的可视化。

**add_figure(tag, figure, global_step=None, close=True, walltime=None)**

 - 功能：将matplotlib的figure绘制到tensorboard中。

**add_histogram(tag, values, global_step=None, bins='tensorflow')**

 - tag：字符串形式的标签，用于区分不同类型的直方图数据。 
 - values：要记录的数据值。
 -  global_step：全局步骤数。
 - bins：直方图的分箱方式，默认为 'tensorflow'。
 - 功能：绘制直方图。这里的global_step表明会得到多个直方图。在tensorboard界面，需要进入HISTOGRAM中才能看到直方图可视化。

**add_graph(model, input_to_model=None, verbose=False)**

 - model：要记录的模型。 
 - input_to_model：模型的输入数据。 
 - verbose：是否输出详细的日志信息。
 - 功能：绘制pytorch模型拓扑结构图。

**add_embedding(mat, metadata=None, label_img=None, global_step=None, tag='default')**

 - mat：嵌入矩阵。 
 - metadata：元数据，如类别标签。 
 - label_img：用于标记的图像数据。 
 - global_step：全局步骤数。
 -   tag：字符串形式的标签。
 - 功能：绘制高维数据在低维的投影

**close()**

 - 关闭 SummaryWriter 实例，释放资源。

```python
add_video：绘制视频

add_audio：绘制音频，可进行音频播放。

add_text：绘制文本

add_pr_curve：绘制PR曲线，二分类任务中很实用。

add_mesh：绘制网格、3D点云图。

add_hparams：记录超参数组，可用于记录本次曲线所对应的超参数。
```

### CNN卷积核与特征图可视化
torchvision.utils.make_grid 是一个用于将多个图像排列成一个网格图像的函数。这个函数在 PyTorch 中非常有用，特别是在需要可视化一批图像的时候，比如在训练神经网络的过程中查看输入图像或中间层的特征图。

```python
torchvision.utils.make_grid(tensor, nrow=8, padding=2, normalize=False, range=None, scale_each=False, pad_value=0, **kwargs)
```
#### 参数说明

```python
tensor (Tensor or list of tensors)：
输入的图像张量或张量列表。如果输入是张量，则形状应为 (batch_size, channels, height, width) 或 (channels, height, width)。
nrow (int, optional)：
每行中图像的数量，默认为 8。
padding (int, optional)：
图像之间的填充像素数量，默认为 2。
normalize (bool, optional)：
是否对图像进行归一化处理，默认为 False。
range (tuple, optional)：
归一化的范围，如果 normalize=True，则使用此参数指定归一化的最小值和最大值。
scale_each (bool, optional)：
是否单独对每个图像进行归一化处理，默认为 False。
pad_value (float, optional)：
填充像素的值，默认为 0。
**kwargs (dict, optional)：
其他可选参数，例如 dtype 等。
```

#### 返回值

```python
Tensor：
返回一个排列后的网格图像张量，形状为 (channels, grid_height, grid_width)。
```
### 混淆矩阵与训练曲线可视化

#### 混淆矩阵可视化

```python
show_conf_mat(confusion_mat, classes, set_name, out_dir, epoch=999, verbose=False, perc=False)
```
    """
    混淆矩阵绘制并保存图片
    :param confusion_mat:  nd.array
    :param classes: list or tuple, 类别名称
    :param set_name: str, 数据集名称 train or valid or test?
    :param out_dir:  str, 图片要保存的文件夹
    :param epoch:  int, 第几个epoch
    :param verbose: bool, 是否打印精度信息
    :param perc: bool, 是否采用百分比，图像分割时用，因分类数目过大
    :return:
    """

#### 训练曲线绘制
loss曲线是需要将训练与验证放在一起看的，单独看一条曲线是不够的。通过训练loss看偏差，通过训练loss与验证loss看方差。偏差看的是模型拟合能力是否足够，方差是看模型泛化性能是否足够，是否存在过拟合。

### 模型参数打印
torchinfo 是一个用于分析 PyTorch 模型结构和计算复杂度的库。它可以方便地展示模型的每一层的信息，包括输入输出形状、参数数量等，对于理解模型架构和优化模型非常有帮助。

#### 参数说明
**model (nn.Module)：**

 - 需要分析的 PyTorch 模型实例。

**input_size (tuple)：**

 - 输入数据的形状。例如 (batch_size, channels, height, width)。

#### 输出解释
torchinfo.summary 会输出模型每一层的信息，包括：

 - Layer name (层名)：模型中的层名。 
 - Output Shape (输出形状)：该层输出的形状。 
 - Param # (参数数量)：该层的参数数量。 
 - Total params (总参数数量)：模型的总参数数量。 
 - Trainable params  (可训练参数数量)：模型中可训练的参数数量。 
 - Non-trainable params (不可训练参数数量)：模型中不可训练的参数数量。

参考：[https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-6/](https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-6/)
参考：[https://pytorch-cn.readthedocs.io/zh/latest/](https://pytorch-cn.readthedocs.io/zh/latest/)
参考：[https://datawhalechina.github.io/thorough-pytorch/](https://datawhalechina.github.io/thorough-pytorch/)
