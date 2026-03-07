---
title: Python的reshape的用法和reshape(1,-1)、reshape(-1,1)
date: 2026-01-02 08:10:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿
在创建DataFrame的时候常常使用reshape来更改数据的列数和行数。

reshape可以用于**numpy库里的ndarray和array结构以及pandas库里面的DataFrame和Series结构**。

![源数据](/img/9bf8edc19ac848099d258d281dd3aa6c.png)


![reshape函数](/img/faf642ec22414597a0e92a497b4399db.png)

**reshape（行，列）可以根据指定的数值将数据转换为特定的行数和列数，这个好理解，就是转换成矩阵。**

**然而，在实际使用中，特别是在运用函数的时候，系统经常会提示是否需要对数据使用reshape(1,-1)或者reshape(-1,1)进行转换，那这两个转换是什么意思呢？难道还有-1行的数据？
来尝试一下：**

 ![使用reshape(-1,1)](/img/0c198cc7aa5e4f7e92dd279bb1a0f077.png)


在使用了reshape（-1，1）之后，数据集似乎变成了一列，这样看起来不明显，把这些数据导出到excel看看：

 ![导出到excel](/img/782399e3a99b48caa4b6d59842f8c184.png)


 ![excel文件里显示的](/img/c6b8dc807b194d2a923910a61dc2753f.png)

在excel里直接变成了一列。

那么reshape(1,-1)呢？也就是直接变成了一行了。

**那这个-1在这里要怎么理解呢？**
跟进numpy库官网的介绍，这里的-1被理解为unspecified value，意思是未指定为给定的。如果只需要特定的行数，列数多少我无所谓，我只需要指定行数，那么列数直接用-1代替就行了，计算机帮我们算赢有多少列，反之亦然。
所以-1在这里应该可以理解为一个正整数通配符，它代替任何整数。
拿刚才的数据来试试看：

![指定一个值为-1时的数据](/img/9063f89862ad4885a4af4ab84ce15b46.png)


 
