---
title: 深度学习Pytorch-Tensor函数
date: 2026-01-05 08:03:00
tags:
  - "深度学习"
  - "PyTorch"
  - "Tensor"
  - "张量"
categories:
  - "深度学习"
---

﻿@[TOC](深度学习Pytorch-Tensor函数)
# [微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487933&idx=1&sn=7bf999a20800e41806cb05b65098bb4f&chksm=ec0c0362db7b8a74a266afc842b45e7b2b53f2e8eb4b6609af105e2cf0b4aae13d03c6e0dc1e&token=1104317395&lang=zh_CN#rd)
# Tensor的三角函数

```bash
torch.acos(input,out=None)#arccos反三角函数中的反余弦
torch.asin(input,out=None)
torch.atan(input,out=None)
torch.atan2(input,input2,out=None)
torch.cos(input,out=None)
torch.cosh(input,out=None)
torch.sin(input,out=None)
torch.sinh(input,out=None)#双曲正弦函数
torch.tan(input,out=None)
torch.tanh(input,out=None)
```

# Tensor中其他的数学函数

```bash
torch.abs()
torch.sigmoid()
torch.sign() 符号函数
torch.reciprocal() 每个元素的倒数
torch.rsqrt() 对每个元素取平方根后再取倒数
torch.neg() 元素取负
torch.lerp(start, end, weight, out=None):对两个张量以start, end做线性插值，将结果返回到输出张量out = start + weight*(end - start) 
torch.addcdiv(tensor, value=1, tensor1, tensor2, out=None):用tensor2对tensor1逐元素相除，然后乘以标量值value并加到tensor上。
torch.addcmul(tensor, value=1, tensor1, tensor2, out=None):用tensor2对tensor1逐元素相乘，并对结果乘以标量值value然后加到tensor，张量形状不需要匹配，但元素数量必须一致。
torch.cumprod(input, dim, out=None) -> Tensor:返回输入沿指定维度的累积积，如输入是一个N元向量，则结果也是一个N元向量，第i个输出元素值为yi = x1 * x2 * x3 * ...* xi 
torch.cumsum(input, dim, out=None) -> Tensor:返回输入沿指定维度的累积和
```

# Tensor中统计学相关的函数（维度，对于二维数据：dim=0 按列，dim=1 按行，默认 dim=1）

```bash
torch().mean()      #返回平均值
torch().sum()        #返回总和
torch().prod()       #计算所有元素的积
torch().max()        #返回最大值
torch().min()         #返回最小值
torch().argmax()   #返回最大值排序的索引值
torch().argmin()   #返回最小值排序的索引值
torch().std()          #返回标准差
torch().var()          #返回方差
torch().median()   #返回中间值
torch().mode()      #返回众数值
torch.histc(input, bins=100, min=0, max=0, out=None) -> Tensor:计算输入张量的直方图。如果min和max都为0,则利用数据中的最大最小值作为边界。
torch().bincount() #返回每个值的频数，只支持一维的tensor
```

```bash
import torch

a = torch.rand(2, 2)#大小2*2
#可以通过维度来完成降维
print(a)
print(torch.sum(a))
print(torch.sum(a, dim=0))
print(torch.sum(a, dim=1))
```

![在这里插入图片描述](/img/9f0ebc691ecb46acbc2f72cc8fecc3da.png)

# Tensor的torch.distributions(分布函数)
**distributions包含可参数化的概率分布和采样函数**
得分函数

 - 强化学习中策略梯度方法的基础

pathwise derivative估计器

 - 变分自动编码器中的重新参数化技巧

![在这里插入图片描述](/img/d94533737fc54b1b989690a28e2bfce7.png)

KL Divergence 相对熵
Transforms
# Tensor中的随机抽样
定义随机种子:在需要生成随机数据的实验中，每次实验都需要生成数据，为了确保每次运行.py文件时，生成的随机数都是固定的。

```bash
torch.manual_seed(seed)
```

定义随机数满足的分布

```bash
torch.normal(means, std, out=None):返回一个张量，包含从给定means, std的离散正态分布中抽取随机数，均值和标准差的形状不须匹配，但每个张量的元素个数须相同
```

# Tensor中的范数运算
## 范数
在泛函分析中，它定义在赋范线性空间中，并满足一定的条件，即1.非负性，2.齐次性，3.三角不等式。
常被用来度量某个向量空间（或矩阵）中的每个向量的长度或大小。
## 0范数/1范数/2范数/p范数/核函数：核范数是矩阵奇异值的和

 - torch.dist(input,other,p=2)计算p范数
 - torch.norm()计算2范数

```bash
import torch

a = torch.rand(2, 1)
b = torch.rand(2, 1)
print(a, b)
print(torch.dist(a, b, p = 1))
print(torch.dist(a, b, p = 2))
print(torch.dist(a, b, p = 3))
```

![在这里插入图片描述](/img/ee3be136216e484fa258f468f3cb6c40.png)

```bash
print(torch.norm(a))
print(torch.norm(a, p=3))
print(torch.norm(a, p='fro'))#核函数
```

![在这里插入图片描述](/img/40cc54433a9b44d9bb075fd39cc48188.png)

# Tensor中的矩阵分解
**常见的矩阵分解**

```bash
LU分解：将矩阵A分解成L（下三角）矩阵和U（上三角）矩阵的乘积
QR分解：将原矩阵分解成一个正交矩阵Q和一个上三角矩阵R的乘积
EVD分解：特征值分解：PCA
SVD分解：奇异值分解：LDA
```

Pytorch中的奇异值分解

```bash
torch.svd()
```


