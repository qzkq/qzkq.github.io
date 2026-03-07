---
title: 深度学习Pytorch-Tensor的属性、算术运算
date: 2026-01-05 08:04:00
tags:
  - "深度学习"
  - "PyTorch"
  - "Tensor"
  - "张量"
categories:
  - "深度学习"
---

﻿@[TOC](深度学习Pytorch-Tensor的属性、算术运算)
# [微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487933&idx=1&sn=7bf999a20800e41806cb05b65098bb4f&chksm=ec0c0362db7b8a74a266afc842b45e7b2b53f2e8eb4b6609af105e2cf0b4aae13d03c6e0dc1e&token=1104317395&lang=zh_CN#rd)
# Tensor的属性
**每一个Tensor对象都有以下几个属性：torch.dtype、torch.device和torch.layout**

**1、torch.dtype属性标识了torch.Tensor的数据类型。**

**2、torch.device属性标识了torch.Tensor对象在创建之后所存储在的设备名称。**
torch.device包含了两种设备类型 ('cpu' 或者 'cuda') ，分别标识将Tensor对象储存于cpu内存或者gpu内存中，同时支持指定设备编号，比如多张gpu，可以通过gpu编号指定某一块gpu。 如果没有指定设备编号，则默认将对象存储于current_device()当前设备中； 
举个例子， 一个torch.Tensor对象构造函数中的设备字段如果填写'cuda'，那等价于填写了'cuda:X'，其中X是函数 torch.cuda.current_device()的返回值。在torch.Tensor对象创建之后，可以通过访问torch.device属性实时访问当前对象所存储在的设备名称。构造torch.device可以通过字符串/字符串和设备编号。

**3、torch.layout属性标识了torch.Tensor在内存中的布局模式。** 
现在， 我们支持了两种内存布局模式 torch.strided (dense Tensors) 和尚处试验阶段的torch.sparse_coo (sparse COO Tensors， 一种经典的稀疏矩阵存储方式)。
torch.strided 跨步存储代表了密集张量的存储布局方式，当然也是最常用最经典的一种布局方式。 每一个strided tensor都有一个与之相连的torch.Storage对象, 这个对象存储着tensor的数据。这些Storage对象为tensor提供了一种多维的， 跨步的(strided)数据视图。这一视图中的strides是一个interger整形列表：这个列表的主要作用是给出当前张量的各个维度的所占内存大小，严格的定义就是，strides中的第k个元素代表了在第k维度下，从一个元素跳转到下一个元素所需要跨越的内存大小。 跨步这个概念有助于提高多种张量运算的效率。

**Tensor创建实例：**

```bash
import torch
a = torch.Tensor([[1, 2],[3, 4]])#定义一个2*2的张量，数值为1,2，3,4
print(a)
Out[]: 
tensor([[1., 2.],
        [3., 4.]])

b = torch.Tensor(2, 2)#制定形状2*2
print(b)
Out[]: 
tensor([[6.2106e-42, 0.0000e+00],
        [       nan,        nan]])
```

**稀疏张量实例：**

```bash
i = torch.tensor([[0, 1, 2], [0, 1, 2]])#非0元素的坐标（0,0）（1,1）（2,2）
v = torch.tensor([1, 2, 3])#非0元素具体的值对应上述坐标
a = torch.sparse_coo_tensor(i, v, (4, 4),
                            dtype=torch.float32,
                            device=dev)
print(a)
Out[]: 
tensor(indices=tensor([[0, 1, 2],
                       [0, 1, 2]]),
       values=tensor([1., 2., 3.]),
       size=(4, 4), nnz=3, layout=torch.sparse_coo)
#转变为稠密张量
i = torch.tensor([[0, 1, 2], [0, 1, 2]])#非0元素的坐标（0,0）（1,1）（2,2）
v = torch.tensor([1, 2, 3])#非0元素具体的值对应上述坐标
a = torch.sparse_coo_tensor(i, v, (4, 4),
                            dtype=torch.float32,
                            device=dev).to_dense()#大小4*4
print(a)
Out[]: 
tensor([[1., 0., 0., 0.],
        [0., 2., 0., 0.],
        [0., 0., 3., 0.],
        [0., 0., 0., 0.]])
```

# Tensor的算术运算

 - 哈达玛积（element wise,对应元素相乘）
 - 二维矩阵乘法运算操作包括torch.mm()、torch.matmul()、@
 - 对于高维的Tensor（dim>2），定义其矩阵乘法仅在最后的两个维度上，要求前面的维度必须保持一致，就像矩阵的索引一样并且运算操作只有torch.matmul()

**算术运算实例**：add加法、sub减法、mul哈达玛积（乘法）、div除法

```bash
import torch
a = torch.rand(2, 3)
b = torch.rand(2, 3)
print(a)
print(b)
print(a + b)
print(a.add(b))
print(torch.add(a, b))
print(a)
print(a.add_(b))#其中，前三种一样，第四种是对 a 进行了修改。
print(a)
```

![在这里插入图片描述](/img/6a36a4cc55a947cb903f356598cd93ef.png)

**矩阵运算**：

```bash
##matmul
a = torch.ones(2, 1)#形状
b = torch.ones(1, 2)
print(a @ b)
print(a.matmul(b))
print(torch.matmul(a, b))
print(torch.mm(a, b))
print(a.mm(b))

##高维tensor
a = torch.ones(1, 2, 3, 4)
b = torch.ones(1, 2, 4, 3)
print(a.matmul(b).shape)
out[]:
torch.Size([1, 2, 3, 3])
```

**幂运算**pow、指数运算exp、对数运算log、开根号sqrt

```bash
a = torch.tensor([1, 2])#数值
print(torch.pow(a, 3))
print(a.pow(3))
print(a**3)
print(a.pow_(3))
print(a)
```

![在这里插入图片描述](/img/aea21d32bafb473ca5bdca7223a6079b.png)

# Pytorch中的in-place操作

 - “就地”操作，即不允许使用临时变量。
 - 也称为原味操作。
 - x=x+y
 - add_、sub_、mul_等等
# Pytorch中的广播机制
 - 广播机制：张量参数可以自动扩展为相同大小
 - 广播机制需要满足两个条件：

1.每个张量至少有一个维度
2.满足右对齐
A.ndim == B.ndim, 并且**A.shape和B.shape对应位置的元素要么相同要么其中一个是1**

```bash
import torch
a = torch.rand(2, 2)
b = torch.rand(1, 2)
# a, 2*1
# b, 1*2
# c, 2*2
# a = torch.rand(2, 1, 1, 3)
# b = torch.rand(4, 2, 3)
# 2*4*2*3
c = a + b
print(a)
print(b)
print(c)
print(c.shape)
```

![在这里插入图片描述](/img/04d508201916401f82000a6338dde9da.png)

# Tensor的取整/取余运算

```bash
.floor()向下取整数
.ceil()向上取整数
.round()四舍五入>=0.5向上取整，<0.5向下取整
.trunc()裁剪，只取整数部分
.frac()只取小数部分
%取余
torch.fmod(a, b)除数除以元素的余数;torch.remainder(a, b)张量a每个元素的除法余数
```

# Tensor的比较运算

```bash
torch.eq(input,other,out=None) #按成员进行等式操作，相同返回True→Tensor
torch.equal(tensor1,tensor2) #如果tensor1和tensor2有相同的size和elements，则为True→ bool
torch.ge(input,other,out=None) #input>=other,逐个元素比较输入张量input是否大于或等于另外的张量或浮点数other。若大于或等于则返回为True，否则返回False。若张量other无法自动扩展成与输入张量input相同尺寸，则返回为False。→Tensor
torch.gt(input,other,out=None) #input>other→Tensor
torch.le(input,other,out=None) #input<=other→Tensor
torch.lt(input,other,out=None) #input<other→Tensor
torch.ne(input,other,out=None) #input!=other不等于→Tensor
```

# Tensor的取前k个大/前k小/第k小的数值及其索引

```bash
torch.sort(input,dim=None,descending=False,out=None)#对目标input进行排序；**维度，对于二维数据：dim=0 按列排序，dim=1 按行排序，默认 dim=1**
torch.topk(input,k,dim=None,largest=True,sorted=True,out=None)#沿着指定维度返回最大k个数值及其索引值
torch.kthvalue(input,k,dim=None,out=None)#沿着指定维度返回第k个最小值及其索引值
```

```bash
##topk，k和维度dim
a = torch.tensor([[2, 4, 3, 1, 5],
                  [2, 3, 5, 1, 4]])
print(a.shape)

print(torch.topk(a, k=1, dim=1, largest=False))

##topk
a = torch.tensor([[2, 4, 3, 1, 5],
                  [2, 3, 5, 1, 4]])
print(a.shape)

print(torch.topk(a, k=2, dim=0, largest=False))
```

![](/img/23a1def8ecd44a918f2df2fada9f9e0c.png)


![](/img/40ef77013fdd444aa44ea66346ebe400.png)

```bash
print(torch.kthvalue(a, k=2, dim=0))
print(torch.kthvalue(a, k=2, dim=1))
```

![在这里插入图片描述](/img/9736c9fe7be740d78d28cb4580326f39.png)

# Tensor判定是否为finite/inf/nan

 - torch.isfinite(tensor)/torch.isinf(tensor)/torch.isnan(tensor)
 - 返回一个标记元素是否为finite/inf/nan的mask张量(有界，无界，nan)
