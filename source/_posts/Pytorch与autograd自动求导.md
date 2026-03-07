---
title: Pytorch与autograd自动求导
date: 2026-01-02 08:19:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Pytorch与autograd自动求导)
# [微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487933&idx=1&sn=7bf999a20800e41806cb05b65098bb4f&chksm=ec0c0362db7b8a74a266afc842b45e7b2b53f2e8eb4b6609af105e2cf0b4aae13d03c6e0dc1e&token=1104317395&lang=zh_CN#rd)
**梯度**的本意是一个向量（矢量），表示某一函数在该点处的方向导数沿着该方向取得最大值，即函数在该点处沿着该方向（此梯度的方向）变化最快，变化率最大（为该梯度的模）。Pytorch与autograd自动求导梯度的本意是一个向量（矢量），表示某一函数在该点处的方向导数沿着该方向取得最大值，即函数在该点处沿着该方向（此梯度的方向）变化最快，变化率最大（为该梯度的模）。

**每个tensor通过requires_grad来设置是否计算梯度**

 - 用来冻结某些层的参数

# 关于Auttograd的几个概念
## 叶子张量（leaf） is_leaf

> （可以看作为叶子节点）： is_leaf 为False的时候,则不是叶子节点, is_leaf为True的时候为叶子节点(或者叶子张量)
> 当requires_grad()为True时将会记录tensor的运算过程并为自动求导做准备，但是并不是每个requires_grad()设为True的值都会在backward的时候得到相应的grad，它还必须为leaf。也就是：
> leaf成为了在 requires_grad()下判断是否需要保留 grad的前提条件

 - 按照惯例,所有requires_grad为False的张量(Tensor) 都为叶子张量( leaf Tensor)
 - requires_grad为True的张量(Tensor),如果他们是由用户创建的,则它们是叶子张量(leaf Tensor).这意味着它们不是运算的结果,因此grad_fn为None
 - 只有是叶张量的tensor在反向传播时才会将本身的grad传入的backward的运算中. 如果想得到当前tensor在反向传播时的grad, 可以用retain_grad()这个属性
## grad VS grad_fn
 - grad:该Tensor的梯度值，每次在计算backward时都需要将前一时刻的梯度归零，否则梯度值会一直增加。
 - grad_fn:叶子节点通常为None，只有结果节点的grad_fn才有效，用于指示梯度函数是哪种类型。
## backward函数:计算那些有关图中叶子节点的tensors的梯度的和

```bash
torch.autograd.backward(tensor,gradtensor=None,retaingraph=None,create_graph=False)
```

 - retain_graph:通常在调用一次backward后，pytorch会自动吧计算图销毁，所以想要对某个变量重复调用backward，则需要将该参数设置为True
 - create_graph:如果为True，那么就创建一个专门的graph of the derivative，这可以方便计算高阶微分。
## torch.autograd.grad()函数

```bash
def grad(outputs, inputs, grad_outputs=None, retain_graph=None,create_graph=False,only_inputs=True, allow_unused=False)
```

 - 计算和返回output关于inputs的梯度的和
 - outputs:函数的因变量，即需要求导的那个函数
 - inputs：函数的自变量
 - grad_outputs:同backward
 - only_inputs:只计算input的梯度
 - allow_unused(bool,可选):如果为False，当计算输出出错时（因此他们的梯度永远是0）指明不使用的inputs。
## torch.autograd包中的其它函数

```bash
torch.autograd.enable_grad:启动梯度计算的上下文管理器
torch.autograd.no_grad:禁止梯度计算的上下文管理器
torch.autograd.set_grad_enabled(mode):设置是否进行梯度计算的上下文管理器
```

## torch.autograd.Function
**每一个原始的自动求导运算实际上是两个在Tensor上运行的函数**

 - forward函数计算从输入Tensors获得的输出Tensors
 - backward函数接收输出Tensors对于某个标量值的梯度，并且计算输入Tensors相对于该相同标量值的梯度
 - 最后，利用apply方法执行相应的运算
 - 		定义在Function类的父类_FunctionBase中定义的一个方法

```bash
import torch

class line(torch.autograd.Function):
    @staticmethod #静态函数
    def forward(ctx, w, x, b):#向前运算
        #y = w*x +b
        ctx.save_for_backward(w, x, b)
        return w * x + b

    @staticmethod
    def backward(ctx, grad_out):#反向传播
        w, x, b = ctx.saved_tensors

        grad_w = grad_out * x
        grad_x = grad_out * w
        grad_b = grad_out

        return grad_w, grad_x, grad_b


w = torch.rand(2, 2, requires_grad=True)
x = torch.rand(2, 2, requires_grad=True)
b = torch.rand(2, 2, requires_grad=True)

out = line.apply(w, x, b)
out.backward(torch.ones(2, 2))

print(w, x, b)
print(w.grad, x.grad, b.grad)
```

![在这里插入图片描述](/img/57f588d86cd643fa849dbcdacd83ac53.png)

