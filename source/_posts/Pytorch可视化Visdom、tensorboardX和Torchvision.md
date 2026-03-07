---
title: Pytorch可视化Visdom、tensorboardX和Torchvision
date: 2026-01-03 08:01:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Pytorch可视化Visdom、tensorboardX和Torchvision)
# [微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487933&idx=1&sn=7bf999a20800e41806cb05b65098bb4f&chksm=ec0c0362db7b8a74a266afc842b45e7b2b53f2e8eb4b6609af105e2cf0b4aae13d03c6e0dc1e&token=1104317395&lang=zh_CN#rd)
# Visdom介绍
## visdom旨在促进(远程)数据的可视化，重点是支持科学实验。pytorch常用可视化工具。 

 - 支持数值（折线图，直方图等）、图像、文本以及视频等
 - 支持Pytroch、Torch和Numpy
 - 用户可以通过编程的方式组织可视化空间或者通过用户接口为数据打造仪表盘，检查实验结果和测试代码。
	 - env:环境 & pane：窗格

**安装：pip install visdom**
**启动服务：python -m visdom.server**

```bash
import visdom
import numpy as np
vis = visdom.Visdom()
vis.text('Hello ,world')
vis.image(np.ones((3,10,10)))
```

# tensorboardX介绍

![在这里插入图片描述](/img/bda37112eda74da5b1ab6aac44ae0c63.png)

```bash
from tensorboardX import SummaryWriter

writer = SummaryWriter("log")
for i in range(100):
    writer.add_scalar("a", i, global_step=i)
    writer.add_scalar("b", i ** 2, global_step=i)
writer.close()
```

命令行运行进入log文件夹下：

 1. cd D:\JetBrains\PycharmProjects\pytorch_code\log
 2. tensorboard --logdir ./
 3. 打开运行结果里的网站（需要安装tensorborad）

# Torchvision介绍
**torchvision是独立于pytorch的关于图像操作的一些方便工具库**

 - https://github.com/pytorch/vision
 - https://pytorch.org/docs/master/torchvision/

**torchvision主要包括以下几个包：**

 - vision.datasets:几个常用视觉数据集，可以下载和加载
 - vision.models:已经训练好的模型，例如：AlexNet，VGG，ResNet
 - vision.transforms:常用的图像操作，例如：随机切割，旋转，数据类型转换，图像到tensor,numpy数组到tensor，tensor到图像等
 - vision.utils、vision.io、vision.ops

