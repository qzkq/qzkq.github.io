---
title: Pytorch_模型的保存_加载、并行化、分布式
date: 2026-01-02 08:18:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Pytorch:模型的保存/加载、并行化、分布式)
# [微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487933&idx=1&sn=7bf999a20800e41806cb05b65098bb4f&chksm=ec0c0362db7b8a74a266afc842b45e7b2b53f2e8eb4b6609af105e2cf0b4aae13d03c6e0dc1e&token=1104317395&lang=zh_CN#rd)
# 模型的保存/加载

 - torch.saves(state,dir) 保存/序列化
 - torch.load(dir) 加载模型
# 并行化

```bash
torch.get_num_threads():
```

 - 获得用于并行化CPU操作的OpenMP线程数

```bash
torch.set_num_threads(int):
```

 - 设定用于并行化CPU操作的OpenMP线程数
# 分布式
 - python在默认情况下只使用一个GPU，再多个GPU的情况下就需要使用pytorch提供的DataParallel
 - 单机多卡
 - 多机多卡
# Tensor on GPU
**用方法to()可以将Tensor在CPU和GPU（需要硬件支持）之间相互移动**

![在这里插入图片描述](/img/c908e2c7868542c2ad20c9915035338e.png)

# Tensor的相关配置

```bash
torch.is_tensor() #如果是pytorch的tensor类型返回true
torch.is_storage() #如果是pytorch的storage类型返回ture
torch.set_flush_denormal(mode) #防止一些不正常的元素产生
torch.set_default_dtype(d) #对torch.tensor()设置默认的浮点类型
torch.set_printoptions(precision=None,threshold=None,edgeitems=None,linewidth=None,profile=None) #设置printing的打印参数
```

# Tensor与numpy的转换

```bash
torch.from_numpy(ndarry)
a.numpy()
```

