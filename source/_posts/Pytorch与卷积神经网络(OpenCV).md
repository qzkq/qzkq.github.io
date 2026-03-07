---
title: Pytorch与卷积神经网络(OpenCV)
date: 2026-01-03 08:00:00
tags:
  - "深度学习"
  - "CNN"
  - "卷积神经网络"
  - "面试"
categories:
  - "深度学习"
---

﻿
@[TOC](Pytorch与卷积神经网络 OpenCV)
# [微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487933&idx=1&sn=7bf999a20800e41806cb05b65098bb4f&chksm=ec0c0362db7b8a74a266afc842b45e7b2b53f2e8eb4b6609af105e2cf0b4aae13d03c6e0dc1e&token=1104317395&lang=zh_CN#rd)
# 卷积神经网：以卷积层为主的深度网络结构

```bash
卷积层
激活层
BN层（Batch Normalization:批量归一化）
池化层
FC层（fully connected layers:全连接层）
损失层
```

## 卷积层的定义
对图像和滤波矩阵做内积（逐个元素相乘再求和）的操作

```bash
Conv2d(in_channels, out_channels, kernel_size, stride=1,padding=0, dilation=1, groups=1,bias=True)
```

```bash
in_channels：输入的通道数目 
out_channels： 输出的通道数目 
kernel_size：卷积核的大小，类型为int或者元组，当卷积是方形的时候，只需要一个整数边长即可，卷积不是方形，要输入一个元组表示高和宽。
stride： 卷积每次滑动的步长为多少，默认是 1 
padding： 设置在所有边界增加值为 0 的边距的大小（也就是在feature map 外围增加几圈 0 ），例如当 padding =1 的时候，如果原来大小为 3 × 3 ，那么之后的大小为 5 × 5 。即在外围加了一圈 0 。
dilation：控制卷积核之间的间距
groups：控制输入和输出之间的连接.
比如 groups 为1，那么所有的输入都会连接到所有输出
当 groups 为 2的时候，相当于将输入分为两组，并排放置两层，每层看到一半的输入通道并产生一半的输出通道，并且两者都是串联在一起的。这也是参数字面的意思：“组” 的含义。
需要注意的是，in_channels 和 out_channels 必须都可以整除 groups，否则会报错
```

## 常见的卷积操作

```bash
标准卷积
分组卷积（group参数）
空洞卷积（dilation参数）
深度可分离卷积（分组卷积+1*1卷积）
反卷积（torch.nn.ConvTranspose2d）
可变形卷积等等
```

## 如何理解卷积层感受野？
感受野（Receptive Field)，指的是神经网络中神经元“看到的”输入区域，在卷积神经网络中，feature map上某个元素的计算受输入图像上某个区域的影响，这个区域即该元素的感受野。
## 如何理解卷积层的参数量与计算量
参数量:参与计算参数的个数占用内存空间
![在这里插入图片描述](/img/b9f4c1b5df9e42bfbbcba33e752d4ae1.png)

FLOPS:每秒浮点运算次数，理解为计算速度。是一个衡量硬件性能的指标。
FLOPs:浮点运算数，理解为计算量。可以用来衡量算法/模型的复杂度。
![在这里插入图片描述](/img/dea2588c5c6046559072b0f6e49927a3.png)
 
MAC:乘加次数，用来衡量计算量。
![在这里插入图片描述](/img/2d253db8b28c49f48446c1947521ee83.png)
## 如何压缩卷积层参数&计算量?
从感受野不变+减少参数量的角度压缩卷积层

```bash
采用多个3×3卷积核代替大卷积核
采用深度可分离卷积
通道Shuffle
Pooling层
Stride=2等等
```

## 池化层
对输入的特征图进行压缩

 - 一方面使特征图变小，简化网络计算复杂度;
 - 一方面进行特征压缩，提取主要特征

最大池化(Max Pooling)、平均池化(Average Pooling)等

```bash
nn.MaxPool2d(kernel_size, stride=None, padding=O, dilation=1,return_indices=False, ceil_mode=False)
```

参数：

```bash
kernel_size(int or tuple) - max pooling的窗口大小，
stride(int or tuple, optional) - max pooling的窗口移动的步长。默认值是kernel_size
padding(int or tuple, optional) - 输入的每一条边补充0的层数
dilation(int or tuple, optional) – 一个控制窗口中元素步幅的参数
return_indices - 如果等于True，会返回输出最大值的序号，对于上采样操作会有帮助
ceil_mode - 如果等于True，计算输出信号大小的时候，会使用向上取整，代替默认的向下取整的操作
```

## 上采样层
Resize，如双线性插值直接缩放，类似于图像缩放，概念可见最邻近插值算法和双线性插值算法——图像缩放
Deconvolution，也叫Transposed Convolution（转置卷积/反卷积）
实现函数

```bash
nn.functional.interpolate(input, size=None, scale_factor=None,mode='nearest", align_corners=None)
```

```bash
nn.ConvTranspose2d(in_channels, out_channels, kernel_size, stride=1,padding=0, output_padding=0, bias=True)
```

## 激活层
激活函数:为了增加网络的非线性，进而提升网络的表达能力
ReLU函数、Leakly ReLU函数、ELU函数等
torch.nn.ReLU(inplace=True)
## BatchNorm层
通过一定的规范化手段，把每层神经网络任意神经元这个输入值的分布强行拉回到均值为0方差为1的标准正态分布
Batchnorm是归一化的一种手段，它会减小图像之间的绝对差异，突出相对差异，加快训练速度
不适用的问题: image-to-image以及对噪声敏感的任务

```bash
nn.BatchNorm2d(num_features, eps=1e-05,momentum=0.1, affine=True,track_running_stats=True)
```

```bash
num_features：一般输入参数为batch_size*num_features*height*width，即为其中特征的数量
eps：分母中添加的一个值，目的是为了计算的稳定性，默认为：1e-5
```

## 全连接层
连接所有的特征，将输出值送给分类器（如softmax分类器)

```bash
对前层的特征进行一个加权和，(卷积层是将数据输入映射到隐层特征空间)
将特征空间通过线性变换映射到样本标记空间（也就是label)
可以通过1×1卷积+global average pooling代替
可以通过全连接层参数冗余
全连接层参数和尺寸相关
nn.Linear(in_features, out_features,bias)
```

## Dropout层
在不同的训练过程中随机扔掉一部分神经元
测试过程中不使用随机失活，所有的神经元都激活
为了防止或减轻过拟合而使用的函数，它一般用在全连接层nn.dropout
## 损失层
损失层:设置一个损失函数用来比较网络的输出和目标值，通过最小化损失来驱动网络的训练
网络的损失通过前向操作计算，网络参数相对于损失函数的梯度则通过反向操作计算
分类问题损失

```bash
nn.BCELoss; nn.CrossEntropyLoss等等
```

回归问题损失

```bash
nn.L1Loss; nn.MSELoss; nn.SmoothL1Loss等等
```

## Attention机制
对于全局信息,注意力机制会重点关注一些特殊的目标区域,也就是所谓的注意力焦点，进而利用有限的注意力资源对信息进行筛选，提高信息处理的准确性和效率
one-hot分布或者soft的软分布
Soft-Attention 或者Hard-Attention
可以作用在特征图上，尺度空间上，channel尺度上，不同时刻历史特征上等
## 学习率
学习率作为监督学习以及深度学习中重要的超参，其决定着目标函数能否收敛到局部最小值以及何时收敛到最小值。
合适的学习率能够使目标函数在合适的时间内收敛到局部最小值

```bash
torch.optim.Ir_scheduler
```

```bash
ExponentialLR
ReduceLROnPlateau
CyclicLR等等
```

## 优化器
GD、BGD、SGD、MBGD

 - 引入了随机性和噪声

Momentum、NAG等

 - 加入动量原则，具有加速梯度下降的作用

AdaGrad，RMSProp，Adam、AdaDelta

 - 自适应学习率

torch.optim.Adam
## 卷积神经网添加正则化
L1正则:参数绝对值的和
L2正则:参数的平方和（Pytorch自带，weight_decay)

```bash
optimizer =
torch.optim.SGD(model.parameters(),Ir=0.01,weight_decay=0.001)
```

# OpenCV及其常用库函数介绍
## 1. 图像读取与显示
### cv2.imread(filepath,flags)读取图像文件

 - filepath：要读入图片的完整路径
 - flags：读入图片的标志

		cv2.IMREAD_COLOR：默认参数，读入一副彩色图片，忽略alpha通道，可用1作为实参替代
		cv2.IMREAD_GRAYSCALE：读入灰度图片，可用0作为实参替代
		cv2.IMREAD_UNCHANGED：读入完整图片，包括alpha通道，可用-1作为实参替代

### cv2.imshow(wname,img)显示图像
第一个参数是显示图像的窗口的名字，第二个参数是要显示的图像（imread读入的图像），窗口大小自动调整为图片大小

		cv2.waitKey:等待键盘输入，单位为毫秒，即等待指定的毫秒数看是否有键盘输入，若在等待时间内按下任意键则返回按键的ASCII码，程序继续运行。若没有按下任何键，超时后返回-1。参数为0表示无限等待。不调用waitKey的话，窗口会一闪而逝，看不到显示的图片。
		cv2.destroyAllWindow()销毁所有窗口
		cv2.destroyWindow(wname)销毁指定窗口
### cv2.imwrite(file，img，num)保存一个图像
第一个参数是要保存的文件名，第二个参数是要保存的图像。可选的第三个参数，它针对特定的格式：对于JPEG，其表示的是图像的质量，用0   - 100的整数表示，默认95;对于png ,第三个参数表示的是压缩级别。默认为3.

## 2. 图像转换
### cv2.cvtColor(p1,p2) 颜色空间转换。
p1是需要转换的图片，p2是转换成何种格式。

		cv2.COLOR_BGR2RGB #灰度图像转为彩色图像
		cv2.COLOR_BGR2GRAY #彩色图像转为灰度图像
### cv2.resize：调整图像大小。

```python
 resized_image = cv2.resize(image, (new_width, new_height))
```

### cv2.flip：图像翻转。

```python
 flipped_image = cv2.flip(image, 1)  # 1 表示水平翻转，0 表示垂直翻转，-1 表示水平和垂直翻转
```
## 3. 图像处理
### cv2.GaussianBlur：高斯模糊。

```python
blurred_image = cv2.GaussianBlur(image, (5, 5), 0)
```
 - cv2.blur(img,ksize) 均值滤波, cv2.GaussianBlur()高斯滤波, cv2.medianBlur()中值滤波,cv2.bilateralFilter()双边滤波
### cv2.Canny：边缘检测。

```python
edges = cv2.Canny(image, 100, 200)
```
### cv2.threshold：阈值处理。

```python
_, binary_image = cv2.threshold(gray_image, 127, 255, cv2.THRESH_BINARY)
```
## 4. 形态学操作
### cv2.erode(src, kernel, iteration) 腐蚀操作。
		参数说明：src表示的是输入图片，kernel表示的是方框的大小，iteration表示迭代的次数, cv2.dilate()膨胀操作, cv2.morphologyEx()开运算、闭运算

### cv2.dilate：膨胀操作。

```python
dilated_image = cv2.dilate(image, kernel, iterations=1)
```
### cv2.morphologyEx：形态学扩展操作（如开运算、闭运算）。

```python
opening = cv2.morphologyEx(image, cv2.MORPH_OPEN, kernel)
closing = cv2.morphologyEx(image, cv2.MORPH_CLOSE, kernel)
```
## 5. 特征检测
### cv2.findContours：查找轮廓。

```python
contours, _ = cv2.findContours(binary_image, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
```
### cv2.drawContours：绘制轮廓。

```python
cv2.drawContours(image, contours, -1, (0, 255, 0), 3)
```
### cv2.goodFeaturesToTrack：角点检测。

```python
corners = cv2.goodFeaturesToTrack(gray_image, maxCorners=100, qualityLevel=0.01, minDistance=10)
corners = np.int0(corners)
for i in corners:
    x, y = i.ravel()
    cv2.circle(image, (x, y), 3, 255, -1)
```
## 6. 直方图
### cv2.calcHist：计算直方图。

```python
hist = cv2.calcHist([image], [0], None, [256], [0, 256])
```
### cv2.equalizeHist：直方图均衡化。

```python
equalized_image = cv2.equalizeHist(gray_image)
```

## 7. 视频处理
### cv2.VideoCapture：读取视频。

```python
cap = cv2.VideoCapture('path/to/video.mp4')

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break
    cv2.imshow('Frame', frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```
### cv2.VideoWriter：保存视频。

```python
fourcc = cv2.VideoWriter_fourcc(*'XVID')
out = cv2.VideoWriter('output_video.avi', fourcc, 20.0, (640, 480))

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break
    out.write(frame)

cap.release()
out.release()
cv2.destroyAllWindows()
```

