---
title: 从零搭建CBAM、SENet、STN、transformer、mobile_vit、simple_vit、vit模型（Pytorch代码示例）
date: 2026-01-04 08:01:00
tags:
  - "深度学习"
  - "注意力机制"
  - "CBAM"
  - "SENet"
  - "STN"
  - "PyTorch"
categories:
  - "深度学习"
---

﻿@[TOC](从零搭建深度学习模型（Pytorch代码示例二）)
## CBAM
CBAM（Convolutional Block Attention Module）是一种注意力机制，可以在现有的卷积神经网络（CNN）中插入，以增强模型对重要特征的关注。CBAM 通过同时考虑通道维度和空间维度的注意力，提高了模型的表征能力和性能。以下是 CBAM 的一些关键特点和实现细节：
### CBAM 的关键特点

 - 通道注意力（Channel Attention）：通过计算每个通道的重要性权重，增强或抑制特定通道的特征。
 - 空间注意力（Spatial Attention）：通过计算每个位置的重要性权重，增强或抑制特定区域的特征。
 - 轻量级：CBAM 可以轻松地插入到现有的 CNN 架构中，而不会显著增加计算复杂度。
### CBAM 的基本结构
CBAM 包括两个主要模块：通道注意力模块和空间注意力模块。
1. 通道注意力模块（Channel Attention Module）
最大池化：对输入特征图进行最大池化操作。
平均池化：对输入特征图进行平均池化操作。
共享多层感知机（MLP）：通过两个全连接层（FC）来学习通道的重要性权重。
Sigmoid 激活：将 MLP 的输出通过 Sigmoid 函数转换为权重。
2. 空间注意力模块（Spatial Attention Module）
最大池化：对输入特征图进行最大池化操作。
平均池化：对输入特征图进行平均池化操作。
卷积层：通过一个 7x7 的卷积层来学习空间的重要性权重。
Sigmoid 激活：将卷积层的输出通过 Sigmoid 函数转换为权重。
```python
import torch
import torch.nn as nn
import math
import torch.utils.model_zoo as model_zoo

model_urls = {
    'resnet18': 'https://download.pytorch.org/models/resnet18-5c106cde.pth',
    'resnet34': 'https://download.pytorch.org/models/resnet34-333f7ec4.pth',
    'resnet50': 'https://download.pytorch.org/models/resnet50-19c8e357.pth',
    'resnet101': 'https://download.pytorch.org/models/resnet101-5d3b4d8f.pth',
    'resnet152': 'https://download.pytorch.org/models/resnet152-b121ed2d.pth',
}


def conv3x3(in_planes, out_planes, stride=1):
    return nn.Conv2d(in_planes, out_planes, kernel_size=3, stride=stride,
                     padding=1, bias=False)

## 通道注意力模块
class ChannelAttention(nn.Module):
    def __init__(self, in_planes, ratio=16):
        super(ChannelAttention, self).__init__()
        self.avg_pool = nn.AdaptiveAvgPool2d(1)
        self.max_pool = nn.AdaptiveMaxPool2d(1)
           
        self.fc = nn.Sequential(nn.Conv2d(in_planes, in_planes // ratio, 1, bias=False),
                               nn.ReLU(),
                               nn.Conv2d(in_planes // ratio, in_planes, 1, bias=False))
        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        avg_out = self.fc(self.avg_pool(x))
        max_out = self.fc(self.max_pool(x))
        out = avg_out + max_out
        return self.sigmoid(out)

## 空间注意力模块
class SpatialAttention(nn.Module):
    def __init__(self, kernel_size=7):
        super(SpatialAttention, self).__init__()

        self.conv1 = nn.Conv2d(2, 1, kernel_size, padding=kernel_size//2, bias=False)
        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        avg_out = torch.mean(x, dim=1, keepdim=True)
        max_out, _ = torch.max(x, dim=1, keepdim=True)
        x = torch.cat([avg_out, max_out], dim=1)
        x = self.conv1(x)
        return self.sigmoid(x)

## ResNet18与ResNet34使用的残差模块
class BasicBlock(nn.Module):
    expansion = 1

    def __init__(self, inplanes, planes, stride=1, downsample=None):
        super(BasicBlock, self).__init__()
        self.conv1 = conv3x3(inplanes, planes, stride)
        self.bn1 = nn.BatchNorm2d(planes)
        self.relu = nn.ReLU(inplace=True)
        self.conv2 = conv3x3(planes, planes)
        self.bn2 = nn.BatchNorm2d(planes)

        self.ca = ChannelAttention(planes) # 通道注意力
        self.sa = SpatialAttention() # 空间注意力

        self.downsample = downsample
        self.stride = stride

    def forward(self, x):
        residual = x

        out = self.conv1(x)
        out = self.bn1(out)
        out = self.relu(out)

        out = self.conv2(out)
        out = self.bn2(out)

        out = self.ca(out) * out
        out = self.sa(out) * out

        if self.downsample is not None:
            residual = self.downsample(x)

        out += residual
        out = self.relu(out)

        return out

## ResNet50,ResNet101与ResNet152使用的残差模块
class Bottleneck(nn.Module):
    expansion = 4

    def __init__(self, inplanes, planes, stride=1, downsample=None):
        super(Bottleneck, self).__init__()
        self.conv1 = nn.Conv2d(inplanes, planes, kernel_size=1, bias=False)
        self.bn1 = nn.BatchNorm2d(planes)
        self.conv2 = nn.Conv2d(planes, planes, kernel_size=3, stride=stride,
                               padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(planes)
        self.conv3 = nn.Conv2d(planes, planes * 4, kernel_size=1, bias=False)
        self.bn3 = nn.BatchNorm2d(planes * 4)
        self.relu = nn.ReLU(inplace=True)

        self.ca = ChannelAttention(planes * 4)
        self.sa = SpatialAttention()

        self.downsample = downsample
        self.stride = stride

    def forward(self, x):
        residual = x

        out = self.conv1(x)
        out = self.bn1(out)
        out = self.relu(out)

        out = self.conv2(out)
        out = self.bn2(out)
        out = self.relu(out)

        out = self.conv3(out)
        out = self.bn3(out)

        out = self.ca(out) * out
        out = self.sa(out) * out

        if self.downsample is not None:
            residual = self.downsample(x)

        out += residual
        out = self.relu(out)

        return out

## 不同残差网络
class ResNet(nn.Module):
    def __init__(self, block, layers, num_classes=1000):
        self.inplanes = 64
        super(ResNet, self).__init__()
        self.conv1 = nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3,
                               bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.relu = nn.ReLU(inplace=True)
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
        self.layer1 = self._make_layer(block, 64, layers[0])
        self.layer2 = self._make_layer(block, 128, layers[1], stride=2)
        self.layer3 = self._make_layer(block, 256, layers[2], stride=2)
        self.layer4 = self._make_layer(block, 512, layers[3], stride=2)
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512 * block.expansion, num_classes)

    def _make_layer(self, block, planes, blocks, stride=1):
        downsample = None
        if stride != 1 or self.inplanes != planes * block.expansion:
            downsample = nn.Sequential(
                nn.Conv2d(self.inplanes, planes * block.expansion,
                          kernel_size=1, stride=stride, bias=False),
                nn.BatchNorm2d(planes * block.expansion),
            )

        layers = []
        layers.append(block(self.inplanes, planes, stride, downsample))
        self.inplanes = planes * block.expansion
        for i in range(1, blocks):
            layers.append(block(self.inplanes, planes))

        return nn.Sequential(*layers)

    def forward(self, x):
        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)
        x = self.maxpool(x)

        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)

        x = self.avgpool(x)
        x = x.view(x.size(0), -1)
        x = self.fc(x)

        return x

if __name__ == '__main__':
    resnet18 = ResNet(BasicBlock, [2, 2, 2, 2])
    resnet34 = ResNet(BasicBlock, [3, 4, 6, 3])
    resnet50 = ResNet(Bottleneck, [3, 4, 6, 3])
    resnet101 = ResNet(Bottleneck, [3, 4, 23, 3])
    resnet152 = ResNet(Bottleneck, [3, 8, 36, 3])

    x = torch.randn(1, 3, 224, 224)
    output = resnet18(x)
    torch.onnx.export(resnet18,x,'resnet18_cbam.onnx')

    # 使用预训练权重
    pretrained_state_dict = model_zoo.load_url(model_urls['resnet18'])
    new_state_dict = resnet18.state_dict()
    new_state_dict.update(pretrained_state_dict)
    resnet18.load_state_dict(new_state_dict)

```

## SENet
SENet（Squeeze-and-Excitation Network）是一种通过引入注意力机制来增强卷积神经网络（CNN）性能的方法。SENet 通过动态地重新校准特征通道的重要性，使得模型能够更好地关注重要的特征，从而提高模型的表征能力和泛化能力。以下是 SENet 的一些关键特点和实现细节：
### SENet 的关键特点

 - Squeeze 操作：通过全局平均池化（Global Average  Pooling）将每个特征通道压缩成一个全局描述符，捕获每个通道的全局信息。
 - Excitation 操作：通过两个全连接层（FC）学习每个通道的重要性权重，这些权重通过 Sigmoid 激活函数转换为 [0, 1]  范围内的值。
 - Scale 操作：将学习到的权重与原始特征图相乘，增强或抑制特定通道的特征。
### SENet 的基本结构
SENet 通过在传统的卷积块中插入 SE 模块来实现注意力机制。SE 模块包括以下步骤：
 - Squeeze 操作：对输入特征图进行全局平均池化，得到每个通道的全局描述符。
 - Excitation 操作：通过两个全连接层学习每个通道的重要性权重。
 - Scale 操作：将学习到的权重与原始特征图相乘，得到增强后的特征图。
```python
#coding:utf8
import torch
import torch.nn as nn

## SE模块搭建
class SELayer(nn.Module):
    def __init__(self, channel,reduction = 16):
        super(SELayer, self).__init__()
        self.avg_pool = nn.AdaptiveAvgPool2d(1)
        self.fc1 = nn.Linear(in_features=channel, out_features=channel // reduction)
        self.relu = nn.ReLU(inplace=True)
        self.fc2 = nn.Linear(in_features=channel // reduction, out_features=channel)
        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        identity = x
        n,c,_,_ = x.size() # [N,C,H,W]
        x = self.avg_pool(x)
        x = x.view(n,c)
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        x = self.sigmoid(x)
        x = x.view(n,c,1,1)
        x = identity * x

        return x

## 残差模块
class ResidualBlock(nn.Module):
    def __init__(self, in_channels, outchannels):
        super(ResidualBlock, self).__init__()

        self.channel_equal_flag = True
        if in_channels == outchannels:
            self.conv1 = nn.Conv2d(in_channels=in_channels, out_channels=outchannels, kernel_size=3, padding=1, stride=1, bias=False)
        else:
            self.conv1x1 = nn.Conv2d(in_channels=in_channels, out_channels=outchannels, kernel_size=1, stride=2, bias=False)
            self.bn1x1 = nn.BatchNorm2d(num_features=outchannels)
            self.channel_equal_flag = False

            self.conv1 = nn.Conv2d(in_channels=in_channels, out_channels=outchannels,kernel_size=3,padding=1, stride=2, bias=False)

        self.bn1 = nn.BatchNorm2d(num_features=outchannels)
        self.relu = nn.ReLU(inplace=True)

        self.conv2 = nn.Conv2d(in_channels=outchannels, out_channels=outchannels, kernel_size=3,padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(num_features=outchannels)

        self.selayer = SELayer(channel=outchannels)

    def forward(self, x):
        identity = x
        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)

        x = self.conv2(x)
        x = self.bn2(x)
        x = self.relu(x)

        if self.channel_equal_flag == True:
            pass
        else:
            identity = self.conv1x1(identity)
            identity = self.bn1x1(identity)
            identity = self.relu(identity)

        x = self.selayer(x) ## 即插即用模块

        out = identity + x
        return out

## SENet-18
class Model(nn.Module):
    def __init__(self, num_classes=1000):
        super(Model, self).__init__()
        # conv1
        self.conv1 = nn.Conv2d(in_channels=3, out_channels=64,kernel_size=7,stride=2, padding=3, bias=False)
        self.bn1 = nn.BatchNorm2d(num_features=64)
        self.relu = nn.ReLU(inplace=True)

        # conv2_x
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2,padding=1)
        self.conv2_1 = ResidualBlock(in_channels=64, outchannels=64)
        self.conv2_2 = ResidualBlock(in_channels=64, outchannels=64)

        # conv3_x
        self.conv3_1 = ResidualBlock(in_channels=64, outchannels=128)
        self.conv3_2 = ResidualBlock(in_channels=128, outchannels=128)

        # conv4_x
        self.conv4_1 = ResidualBlock(in_channels=128, outchannels=256)
        self.conv4_2 = ResidualBlock(in_channels=256, outchannels=256)

        # conv5_x
        self.conv5_1 = ResidualBlock(in_channels=256, outchannels=512)
        self.conv5_2 = ResidualBlock(in_channels=512, outchannels=512)

        # avg_pool
        self.avg_pool = nn.AdaptiveAvgPool2d(output_size=(1,1)) # [N, C, H, W] = [N, 512, 1, 1]

        # fc
        self.fc = nn.Linear(in_features=512, out_features=num_classes)  # [N, num_classes]

    def forward(self, x):
        # conv1
        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)

        # conv2_x
        x = self.maxpool(x)
        x = self.conv2_1(x)
        x = self.conv2_2(x)

        # conv3_x
        x = self.conv3_1(x)
        x = self.conv3_2(x)

        # conv4_x
        x = self.conv4_1(x)
        x = self.conv4_2(x)

        # conv5_x
        x = self.conv5_1(x)
        x = self.conv5_2(x)

        # avgpool + fc
        x = self.avg_pool(x)

        x = torch.flatten(x, 1)

        x = self.fc(x)

        return x


if __name__ == '__main__':
    # 定义模型
    model = Model(num_classes=10)

    # 定义输入 [N, C, H, W]
    input = torch.ones([10, 3, 224, 224])
    output = model(input)
    torch.onnx.export(model,input,'senet.onnx')


```

## STN
STN（Spatial Transformer Networks）是一种用于在卷积神经网络（CNN）中进行空间变换的技术。STN 可以动态地对输入图像进行空间变换，如平移、旋转、缩放等，从而提高模型的鲁棒性和泛化能力。STN 通过引入一个可学习的空间变换模块，使得模型能够自适应地调整输入图像的位置和姿态，从而更好地捕捉特征。
### STN 的关键特点

 - 局部化网络（Localization Network）：用于预测变换参数的网络，通常是一个小的 CNN。
 - 网格生成器（Grid Generator）：根据预测的变换参数生成采样网格。
 - 采样器（Sampler）：根据生成的采样网格对输入图像进行采样，生成变换后的图像。
### STN 的基本结构
STN 的基本结构包括三个主要部分：
 - 局部化网络：接收输入图像并输出变换参数。
 - 网格生成器：根据变换参数生成采样网格。
 - 采样器：根据采样网格对输入图像进行采样，生成变换后的图像。
```python
import torch
from torch import nn
from torch.nn import functional as F

class STN(nn.Module):
    def __init__(self, c,h,w,mode='stn'):
        assert mode in ['stn', 'cnn']

        super(STN, self).__init__()
        self.mode = mode
        self.local_net = LocalNetwork(c,h,w)
        self.conv = nn.Sequential(
            nn.Conv2d(in_channels=3, out_channels=16, kernel_size=3, stride=2, padding=1),
            nn.ReLU(),
            nn.Conv2d(in_channels=16, out_channels=16, kernel_size=3, stride=2, padding=1),
            nn.ReLU(),
        )

        self.fc = nn.Sequential(
            nn.Linear(in_features=16*8*8, out_features=1024),
            nn.ReLU(),
            nn.Linear(in_features=1024, out_features=10)
        )

    def forward(self, img):
        '''
        :param img: (b, c, h, w)
        :return: (b, c, h, w), (b,)
        '''
        batch_size,c,h,w = img.shape
        img = self.local_net(img)

        conv_output = self.conv(img).view(batch_size, -1)
        predict = self.fc(conv_output)
        return img, predict


class LocalNetwork(nn.Module):
    def __init__(self,c,h,w):
        super(LocalNetwork, self).__init__()
        self.fc = nn.Sequential(
            nn.Linear(in_features=c*h*w,
                      out_features=20),
            nn.Tanh(),
            nn.Linear(in_features=20, out_features=6),
            nn.Tanh(),
        )

    def forward(self, img):
        '''
        :param img: (b, c, h, w)
        :return: (b, c, h, w)
        '''
        batch_size,c,w,h = img.shape

        theta = self.fc(img.view(batch_size, -1)).view(batch_size, 2, 3)

        ## 仿射变换采样函数
        grid = F.affine_grid(theta, torch.Size((batch_size,c,h,w)))
        img_transform = F.grid_sample(img, grid)

        return img_transform


if __name__ == '__main__':
    net = STN(3, 32, 32)
    x = torch.randn(1, 3, 32, 32)

    feature,predict = net(x)

    print(feature.shape)


```
## transformer
Transformer 是一种基于自注意力机制（Self-Attention Mechanism）的深度学习模型，最初由 Vaswani 等人在 2017 年的论文《Attention is All You Need》中提出。Transformer 模型在自然语言处理（NLP）任务中取得了显著的成功，尤其是在机器翻译、文本生成、问答系统等领域。以下是 Transformer 的一些关键特点和实现细节：
### Transformer 的关键特点

 - 自注意力机制（Self-Attention Mechanism）：允许模型在处理序列数据时，关注序列中的不同位置，从而捕捉长距离依赖关系。
 - 前馈神经网络（Feed-Forward Neural Network）：每个位置的特征经过相同的前馈神经网络进行处理。
 - 位置编码（Positional Encoding）：由于 Transformer 模型本身不包含序列顺序信息，因此需要添加位置编码来保留序列的顺序信息。
 - 多头注意力机制（Multi-Head Attention）：通过多个不同的注意力头来捕捉不同类型的依赖关系，提高模型的表达能力。
### Transformer 的基本结构
Transformer 模型主要由编码器（Encoder）和解码器（Decoder）组成。每个编码器和解码器都包含多个相同的层。
1. 编码器（Encoder）
多头自注意力机制（Multi-Head Self-Attention）：对输入序列进行自注意力计算。
前馈神经网络（Feed-Forward Neural Network）：对每个位置的特征进行非线性变换。
残差连接（Residual Connections）：在每个子层之后添加残差连接，以缓解梯度消失问题。
层归一化（Layer Normalization）：在每个子层之后进行层归一化，以稳定训练过程。
2. 解码器（Decoder）
多头自注意力机制（Multi-Head Self-Attention）：对目标序列进行自注意力计算。
多头编码器-解码器注意力机制（Multi-Head Encoder-Decoder Attention）：对编码器的输出和目标序列进行交叉注意力计算。
前馈神经网络（Feed-Forward Neural Network）：对每个位置的特征进行非线性变换。
残差连接（Residual Connections）：在每个子层之后添加残差连接。
层归一化（Layer Normalization）：在每个子层之后进行层归一化。
```python
# coding:utf8
# 第一步：导入需要的库
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
import math, copy

import matplotlib.pyplot as plt

# 第二步：定义Transformer类,标准的Encoder-Decoder架构
class Transformer(nn.Module):
    def __init__(self, encoder, decoder, src_embed, tgt_embed, generator):
        super(Transformer, self).__init__()
        # encoder和decoder都是构造的时候传入的，这样会非常灵活
        self.encoder = encoder
        self.decoder = decoder
        # 输入和输出的embedding
        self.src_embed = src_embed
        self.tgt_embed = tgt_embed
        # Decoder部分最后的Linear+softmax
        self.generator = generator

    def forward(self, src, tgt, src_mask, tgt_mask):
        # 接收并处理屏蔽src和目标序列
        # 首先调用encode方法对输入进行编码，然后调用decode方法进行解码
        return self.decode(self.encode(src, src_mask), src_mask, tgt, tgt_mask)

    def encode(self, src, src_mask):
        # 传入的参数包括src的embedding和src_mask
        return self.encoder(self.src_embed(src), src_mask)

    def decode(self, memory, src_mask, tgt, tgt_mask):
        # 传入的参数包括目标的embedding，Encoder的输出memory，及两种掩码
        return self.decoder(self.tgt_embed(tgt), memory, src_mask, tgt_mask)

# 第三步：创建Generator类，最终的输出层,全连接（linear）+ softmax,根据Decoder的隐状态输出一个词
class Generator(nn.Module):
    """d_model是Decoder输出的大小，vocab是词典大小"""

    def __init__(self, d_model, vocab):
        super(Generator, self).__init__()
        self.proj = nn.Linear(d_model, vocab)

    # 全连接再加上一个softmax
    def forward(self, x):
        return F.log_softmax(self.proj(x), dim=-1)

# 第四步：创建LayerNorm类，SublayerConnection类，Feedforward类
class LayerNorm(nn.Module):
    def __init__(self, features, eps=1e-6):
        super(LayerNorm, self).__init__()
        # pytorch中各层权重的数据类型是nn.Parameter，而不是Tensor。故需对初始化后参数（Tensor型）进行类型转换。
        self.a_2 = nn.Parameter(torch.ones(features))
        self.b_2 = nn.Parameter(torch.zeros(features))
        self.eps = eps

    def forward(self, x):
        mean = x.mean(-1, keepdim=True)
        std = x.std(-1, keepdim=True)
        return self.a_2 * (x - mean) / (std + self.eps) + self.b_2

# 不管是Self-Attention还是全连接层，都首先是LayerNorm，然后是Self-Attention/Dense，然后是Dropout，最后是残差连接。这里把它封装成SublayerConnection
class SublayerConnection(nn.Module):
    """
    LayerNorm + sublayer(Self-Attenion/Dense) + dropout + 残差连接
    为了简单，把LayerNorm放到了前面，这和原始论文稍有不同，原始论文LayerNorm在最后
    """
    def __init__(self, size, dropout):
        super(SublayerConnection, self).__init__()
        self.norm = LayerNorm(size)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, sublayer):
        # 将残差连接应用于具有相同大小的任何子层
        return x + self.dropout(sublayer(self.norm(x)))

# Feedforward层
class PositionwiseFeedForward(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super(PositionwiseFeedForward, self).__init__()
        self.w_1 = nn.Linear(d_model, d_ff)
        self.w_2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x):
        return self.w_2(self.dropout(F.relu(self.w_1(x))))

# 第五步：构建HeadedAttention，Scaled Dot Product Attention
def attention(query, key, value, mask=None, dropout=None):
    d_k = query.size(-1) # 特征维度
    # 矩阵(-1,-1,n,d_k)与矩阵(-1,-1,d_k,n)相乘，得到大小为(-1,-1,n,n)的矩阵，n为输入词的长度
    scores = torch.matmul(query, key.transpose(-2, -1)) / math.sqrt(d_k)
    if mask is not None: # 掩码为0的地方
        scores = scores.masked_fill(mask == 0, -1e9)
    p_attn = F.softmax(scores, dim=-1)
    if dropout is not None:
        p_attn = dropout(p_attn)
    return torch.matmul(p_attn, value), p_attn # 矩阵(-1,-1,n,n)与矩阵(-1,-1,n,d_k)相乘，得到大小为(-1,-1,n,d_k)的矩阵

# 计算MultiHeadedAttention，传入head个数及所有head拼接后的model维度
class MultiHeadedAttention(nn.Module):
    def __init__(self, h, d_model, dropout=0.1):
        super(MultiHeadedAttention, self).__init__()
        assert d_model % h == 0
        # 这里假设d_v=d_k
        self.d_k = d_model // h  # 计算每一个头的输入维度，如512/8=64
        self.h = h
        self.linears = clones(nn.Linear(d_model, d_model), 4)  ## 定义线性变换矩阵wq,wk,wv
        self.attn = None
        self.dropout = nn.Dropout(p=dropout)

    def forward(self, query, key, value, mask=None):
        if mask is not None:
            # 相同的mask适应所有的head.
            mask = mask.unsqueeze(1)
        nbatches = query.size(0)

        # 1) 首先使用线性变换，然后把d_model分配给h个Head，每个head为d_k=d_model/h，矩阵格式(nbatches, self.h, n, self.d_k)
        query, key, value = \
            [l(x).view(nbatches, -1, self.h, self.d_k).transpose(1, 2)
             for l, x in zip(self.linears, (query, key, value))]

        # 2) 使用attention函数计算scaled-Dot-product-attention
        # Q=矩阵(nbatches, self.h, n, self.d_k),K=矩阵(nbatches, self.h, self.d_k,n)相乘，
        # A=矩阵(nbatches, self.h, n, n),V=矩阵(nbatches, self.h,n, self.d_k),B=矩阵(nbatches, self.h, n, self.d_k)
        x, self.attn = attention(query, key, value, mask=mask,
                                 dropout=self.dropout)

        # 3) 实现Multi-head attention，用view函数把8个head的64维向量拼接成一个512的向量。
        # 然后再使用一个线性变换(512,512)，shape不变.
        x = x.transpose(1, 2).contiguous().view(nbatches, -1, self.h * self.d_k)
        return self.linears[-1](x)

# 第六步：创建Encoder,Encoder是N个EncoderLayer的堆积而成
def clones(module, N):
    "克隆N个完全相同的SubLayer，使用了copy.deepcopy"
    return nn.ModuleList([copy.deepcopy(module) for _ in range(N)])

class Encoder(nn.Module):
    def __init__(self, layer, N):
        super(Encoder, self).__init__()
        # clone N个layer
        self.layers = clones(layer, N)
        # 再加一个LayerNorm层
        self.norm = LayerNorm(layer.size)

    def forward(self, x, mask):
        for layer in self.layers:
            x = layer(x, mask)
        return self.norm(x)  # N个EncoderLayer处理完成之后还需要一个LayerNorm

# 创建EncoderLayer，由self-attn and feed forward构成
class EncoderLayer(nn.Module):
    def __init__(self, size, self_attn, feed_forward, dropout):
        super(EncoderLayer, self).__init__()
        self.self_attn = self_attn
        self.feed_forward = feed_forward
        # 第一个SublayerConnection是multi attention模块，第一个SublayerConnection是feed forward模块
        self.sublayer = clones(SublayerConnection(size, dropout), 2)
        self.size = size

    def forward(self, x, mask):
        x = self.sublayer[0](x, lambda x: self.self_attn(x, x, x, mask))
        return self.sublayer[1](x, self.feed_forward)

# 第七步：定义Decoder，构建N个完全相同的Decoder层
class Decoder(nn.Module):
    def __init__(self, layer, N):
        super(Decoder, self).__init__()
        self.layers = clones(layer, N)
        self.norm = LayerNorm(layer.size)

    def forward(self, x, memory, src_mask, tgt_mask):
        for layer in self.layers:
            x = layer(x, memory, src_mask, tgt_mask)
        return self.norm(x)

# 创建DecoderLayer，由self-attn, src-attn和feed forward构成
class DecoderLayer(nn.Module):
    def __init__(self, size, self_attn, src_attn, feed_forward, dropout):
        super(DecoderLayer, self).__init__()
        self.size = size
        self.self_attn = self_attn
        self.src_attn = src_attn
        self.feed_forward = feed_forward
        self.sublayer = clones(SublayerConnection(size, dropout), 3)

    def forward(self, x, memory, src_mask, tgt_mask):
        m = memory
        x = self.sublayer[0](x, lambda x: self.self_attn(x, x, x, tgt_mask))
        x = self.sublayer[1](x, lambda x: self.src_attn(x, m, m, src_mask))
        return self.sublayer[2](x, self.feed_forward)

# 构建上三角掩膜矩阵
def subsequent_mask(size):
    "Mask out subsequent positions."
    attn_shape = (1, size, size)
    subsequent_mask = np.triu(np.ones(attn_shape), k=1).astype('uint8')
    return torch.from_numpy(subsequent_mask) == 0

#----------绘制掩膜矩阵图------------#
plt.figure(figsize=(5,5))
plt.imshow(subsequent_mask(6)[0])
plt.savefig('mask.png')

# 第八步：输入数据
# 词嵌入
class Embeddings(nn.Module):
    def __init__(self, d_model, vocab):
        super(Embeddings, self).__init__()
        self.lut = nn.Embedding(vocab, d_model) #vocab为词表大小
        self.d_model = d_model

    def forward(self, x):
        return self.lut(x) * math.sqrt(self.d_model)

# 位置编码
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, dropout, max_len=5000):
        super(PositionalEncoding, self).__init__()
        self.dropout = nn.Dropout(p=dropout)
        # 在对数空间中计算，保证数值稳定性
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2) *
                             -(math.log(10000.0) / d_model))
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)
        self.register_buffer('pe', pe)

    def forward(self, x):
        x = x + self.pe[:, :x.size(1)].clone().detach()
        return self.dropout(x)

#----------绘制位置编码图------------#
## 语句长度为100，假设d_model=20，
plt.figure(figsize=(15, 5))
pe = PositionalEncoding(20, 0)
y = pe.forward(torch.zeros(1, 100, 20))  

plt.plot(np.arange(100), y[0, :, :].data.numpy())
plt.legend(["dim %d"%p for p in [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19]])

plt.savefig('positioncode.png')

# 第九步：构建完整网络
def make_model(src_vocab, tgt_vocab, N=6, d_model=512, d_ff=2048, h=8, dropout=0.1):
    c = copy.deepcopy
    attn = MultiHeadedAttention(h, d_model)
    ff = PositionwiseFeedForward(d_model, d_ff, dropout)
    position = PositionalEncoding(d_model, dropout)
    model = Transformer(
        Encoder(EncoderLayer(d_model, c(attn), c(ff), dropout), N),
        Decoder(DecoderLayer(d_model, c(attn), c(attn),
                             c(ff), dropout), N),
        nn.Sequential(Embeddings(d_model, src_vocab), c(position)),
        nn.Sequential(Embeddings(d_model, tgt_vocab), c(position)),
        Generator(d_model, tgt_vocab))
    return model

# 测试一个简单模型，输入、目标语句长度分别为10，Encoder、Decoder各2层。
if __name__ == '__main__':
    model = make_model(11, 11, N=2)
    src = torch.LongTensor([[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]]) # 本质上是一个索引向量
    src_mask = torch.ones(1, 1, 10)
    torch.onnx._export(model, (src, src, src_mask, src_mask), 'transformer.onnx')
    print(model)

```
## mobile_vit
MobileViT 是一种轻量级的视觉 Transformer 模型，旨在在移动设备上高效运行。它结合了卷积神经网络（CNN）和 Transformer 的优点，通过引入一种新的模块——MobileViT Block，实现了在保持高性能的同时降低计算复杂度。MobileViT 在图像分类、目标检测和语义分割等任务中表现出色，特别适合资源受限的设备。
### MobileViT 的关键特点

 - MobileViT Block：结合了卷积和 Transformer 的优点，通过局部和全局信息的融合来提高模型的表达能力。
 - 轻量级设计：通过使用高效的卷积操作和 Transformer 结构，使得模型在保持高性能的同时具有较低的计算复杂度。
 - 多尺度特征提取：通过多尺度的特征提取，提高了模型对不同尺度特征的捕捉能力。
### MobileViT 的基本结构
MobileViT 主要由以下几个部分组成：
 - Stem：初始的卷积层，用于提取低级别的特征。
 - MobileViT Blocks：核心模块，结合了卷积和 Transformer 的优点。
 - Global Pooling：全局池化层，用于将特征图降维。
 - Classification Head：最终的分类头，用于输出分类结果。
### MobileViT Block 的详细结构
MobileViT Block 是 MobileViT 的核心模块，其结构如下：
 - Local Representation：通过卷积操作提取局部特征。
 - Global Representation：通过 Transformer 提取全局特征。
 - Fusion：将局部特征和全局特征进行融合。

具体步骤如下：
局部特征提取：

 - 使用卷积层提取局部特征。

全局特征提取：

 - 将局部特征展平并重塑为序列形式。
 - 使用 Transformer 进行全局特征提取。

特征融合：

 - 将全局特征重塑回特征图形式。
 - 与局部特征进行融合。

```python
import torch
import torch.nn as nn

from einops import rearrange
from einops.layers.torch import Reduce

# helpers
def conv_1x1_bn(inp, oup):
    return nn.Sequential(
        nn.Conv2d(inp, oup, 1, 1, 0, bias=False),
        nn.BatchNorm2d(oup),
        nn.SiLU()
    )

def conv_nxn_bn(inp, oup, kernel_size=3, stride=1):
    return nn.Sequential(
        nn.Conv2d(inp, oup, kernel_size, stride, 1, bias=False),
        nn.BatchNorm2d(oup),
        nn.SiLU()
    )

class PreNorm(nn.Module):
    def __init__(self, dim, fn):
        super().__init__()
        self.norm = nn.LayerNorm(dim)
        self.fn = fn

    def forward(self, x, **kwargs):
        return self.fn(self.norm(x), **kwargs)

class FeedForward(nn.Module):
    def __init__(self, dim, hidden_dim, dropout=0.):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(dim, hidden_dim),
            nn.SiLU(),
            nn.Dropout(dropout),
            nn.Linear(hidden_dim, dim),
            nn.Dropout(dropout)
        )

    def forward(self, x):
        return self.net(x)

class Attention(nn.Module):
    def __init__(self, dim, heads=8, dim_head=64, dropout=0.):
        super().__init__()
        inner_dim = dim_head * heads
        self.heads = heads
        self.scale = dim_head ** -0.5

        self.attend = nn.Softmax(dim=-1)
        self.dropout = nn.Dropout(dropout)

        self.to_qkv = nn.Linear(dim, inner_dim * 3, bias=False)

        self.to_out = nn.Sequential(
            nn.Linear(inner_dim, dim),
            nn.Dropout(dropout)
        )

    def forward(self, x):
        qkv = self.to_qkv(x).chunk(3, dim=-1)
        q, k, v = map(lambda t: rearrange(
            t, 'b p n (h d) -> b p h n d', h=self.heads), qkv)

        dots = torch.matmul(q, k.transpose(-1, -2)) * self.scale

        attn = self.attend(dots)
        attn = self.dropout(attn)

        out = torch.matmul(attn, v)
        out = rearrange(out, 'b p h n d -> b p n (h d)')
        return self.to_out(out)

class Transformer(nn.Module):
    def __init__(self, dim, depth, heads, dim_head, mlp_dim, dropout=0.):
        super().__init__()
        self.layers = nn.ModuleList([])
        for _ in range(depth):
            self.layers.append(nn.ModuleList([
                PreNorm(dim, Attention(dim, heads, dim_head, dropout)),
                PreNorm(dim, FeedForward(dim, mlp_dim, dropout))
            ]))

    def forward(self, x):
        for attn, ff in self.layers:
            x = attn(x) + x
            x = ff(x) + x
        return x

# MobileNetV2 Block
class MV2Block(nn.Module):
    def __init__(self, inp, oup, stride=1, expansion=4):
        super().__init__()
        self.stride = stride
        assert stride in [1, 2]

        hidden_dim = int(inp * expansion)
        self.use_res_connect = self.stride == 1 and inp == oup

        if expansion == 1:
            self.conv = nn.Sequential(
                # dw
                nn.Conv2d(hidden_dim, hidden_dim, 3, stride,
                          1, groups=hidden_dim, bias=False),
                nn.BatchNorm2d(hidden_dim),
                nn.SiLU(),
                # pw-linear
                nn.Conv2d(hidden_dim, oup, 1, 1, 0, bias=False),
                nn.BatchNorm2d(oup),
            )
        else:
            self.conv = nn.Sequential(
                # pw
                nn.Conv2d(inp, hidden_dim, 1, 1, 0, bias=False),
                nn.BatchNorm2d(hidden_dim),
                nn.SiLU(),
                # dw
                nn.Conv2d(hidden_dim, hidden_dim, 3, stride,
                          1, groups=hidden_dim, bias=False),
                nn.BatchNorm2d(hidden_dim),
                nn.SiLU(),
                # pw-linear
                nn.Conv2d(hidden_dim, oup, 1, 1, 0, bias=False),
                nn.BatchNorm2d(oup),
            )

    def forward(self, x):
        out = self.conv(x)
        if self.use_res_connect:
            out = out + x
        return out

class MobileViTBlock(nn.Module):
    def __init__(self, dim, depth, channel, kernel_size, patch_size, mlp_dim, dropout=0.):
        super().__init__()
        self.ph, self.pw = patch_size

        self.conv1 = conv_nxn_bn(channel, channel, kernel_size)
        self.conv2 = conv_1x1_bn(channel, dim)

        self.transformer = Transformer(dim, depth, 4, 8, mlp_dim, dropout)

        self.conv3 = conv_1x1_bn(dim, channel)
        self.conv4 = conv_nxn_bn(2 * channel, channel, kernel_size)

    def forward(self, x):
        y = x.clone()

        # Local representations
        x = self.conv1(x)
        x = self.conv2(x)

        # Global representations
        _, _, h, w = x.shape
        x = rearrange(x, 'b d (h ph) (w pw) -> b (ph pw) (h w) d',
                      ph=self.ph, pw=self.pw)
        x = self.transformer(x)
        x = rearrange(x, 'b (ph pw) (h w) d -> b d (h ph) (w pw)',
                      h=h//self.ph, w=w//self.pw, ph=self.ph, pw=self.pw)

        # Fusion
        x = self.conv3(x)
        x = torch.cat((x, y), 1)
        x = self.conv4(x)
        return x

# MobileViT模型定义
class MobileViT(nn.Module):
    def __init__(
        self,
        image_size,
        dims,
        channels,
        num_classes,
        expansion=4,
        kernel_size=3,
        patch_size=(2, 2),
        depths=(2, 4, 3)
    ):
        super().__init__()
        assert len(dims) == 3, 'dims must be a tuple of 3'
        assert len(depths) == 3, 'depths must be a tuple of 3'

        ih, iw = image_size
        ph, pw = patch_size
        assert ih % ph == 0 and iw % pw == 0

        init_dim, *_, last_dim = channels

        self.conv1 = conv_nxn_bn(3, init_dim, stride=2)

        self.stem = nn.ModuleList([])
        self.stem.append(MV2Block(channels[0], channels[1], 1, expansion))
        self.stem.append(MV2Block(channels[1], channels[2], 2, expansion))
        self.stem.append(MV2Block(channels[2], channels[3], 1, expansion))
        self.stem.append(MV2Block(channels[2], channels[3], 1, expansion))

        self.trunk = nn.ModuleList([])
        self.trunk.append(nn.ModuleList([
            MV2Block(channels[3], channels[4], 2, expansion),
            MobileViTBlock(dims[0], depths[0], channels[5],
                           kernel_size, patch_size, int(dims[0] * 2))
        ]))

        self.trunk.append(nn.ModuleList([
            MV2Block(channels[5], channels[6], 2, expansion),
            MobileViTBlock(dims[1], depths[1], channels[7],
                           kernel_size, patch_size, int(dims[1] * 4))
        ]))

        self.trunk.append(nn.ModuleList([
            MV2Block(channels[7], channels[8], 2, expansion),
            MobileViTBlock(dims[2], depths[2], channels[9],
                           kernel_size, patch_size, int(dims[2] * 4))
        ]))

        self.to_logits = nn.Sequential(
            conv_1x1_bn(channels[-2], last_dim),
            Reduce('b c h w -> b c', 'mean'),
            nn.Linear(channels[-1], num_classes, bias=False)
        )

    def forward(self, x):
        x = self.conv1(x)

        for conv in self.stem:
            x = conv(x)

        for conv, attn in self.trunk:
            x = conv(x)
            x = attn(x)

        return self.to_logits(x)

if __name__ == '__main__':
    mbvit_xs = MobileViT(
        image_size=(256, 256),
        dims=[96, 120, 144],
        channels=[16, 32, 48, 48, 64, 64, 80, 80, 96, 96, 384],
        num_classes=1000
    )

    img = torch.randn(1, 3, 256, 256)
    pred = mbvit_xs(img)
    print(pred.shape)
    torch.onnx.export(mbvit_xs, img, 'mbvit_xs.onnx')
```
## simple_vit
SimpleViT 是一个简化版的 Vision Transformer (ViT) 模型，旨在降低原始 ViT 的复杂性和计算成本，同时保持一定的性能。ViT 模型的核心思想是将图像视为一系列的 patch，然后通过 Transformer 架构进行处理。SimpleViT 通常会减少层数、隐藏维度或其他参数，以适应更小的计算资源或更快的推理速度。
### SimpleViT 的关键特点

 - 简化架构：减少了原始 ViT 的层数和隐藏维度，降低了模型的复杂度。
 - 高效性：适合在资源受限的环境中运行，如移动设备或嵌入式系统。
 - 易于实现：代码相对简单，便于理解和修改。
### SimpleViT 的基本结构
SimpleViT 主要由以下几个部分组成：
 - Patch Embedding：将图像分割成一系列 patch，并将每个 patch 转换为嵌入向量。
 - Positional Encoding：为每个 patch 添加位置信息，以便模型能够理解 patch 的顺序。
 - Transformer Encoder：通过多个 Transformer 编码器层对 patch 嵌入进行处理。
 - Classification Head：最终的分类头，用于输出分类结果。
### Patch Embedding
Patch Embedding 将图像分割成一系列固定大小的 patch，并将每个 patch 转换为一个嵌入向量。具体步骤如下：
 - 分割图像：将图像分割成一系列固定大小的 patch。
 - 线性变换：将每个 patch 展平并通过线性变换转换为嵌入向量。
 - 添加类别标记：在嵌入向量序列的开头添加一个类别标记（cls token），用于最终的分类任务。
### Positional Encoding
由于 Transformer 模型本身不包含序列顺序信息，因此需要添加位置编码来保留 patch 的顺序信息。位置编码可以通过正弦和余弦函数生成。
### Transformer Encoder
Transformer Encoder 包含多个相同的编码器层，每个编码器层包含多头自注意力机制和前馈神经网络。具体步骤如下：
 - 多头自注意力机制：对 patch 嵌入进行自注意力计算。
 - 前馈神经网络：对每个位置的特征进行非线性变换。
 - 残差连接：在每个子层之后添加残差连接，以缓解梯度消失问题。
 - 层归一化：在每个子层之后进行层归一化，以稳定训练过程。
### Classification Head
Classification Head 通过一个线性变换将最终的类别标记嵌入转换为分类结果。具体步骤如下：
 - 获取类别标记嵌入：从 Transformer Encoder 的输出中获取类别标记嵌入。
 - 线性变换：将类别标记嵌入通过线性变换转换为分类结果。
```python
import torch
from torch import nn

from einops import rearrange
from einops.layers.torch import Rearrange

def pair(t):
    return t if isinstance(t, tuple) else (t, t)

## 位置编码向量
def posemb_sincos_2d(patches, temperature = 10000, dtype = torch.float32):
    _, h, w, dim, device, dtype = *patches.shape, patches.device, patches.dtype
    # 假设h=8,w=8,dim=1024
    y, x = torch.meshgrid(torch.arange(h, device = device), torch.arange(w, device = device), indexing = 'ij')
    assert (dim % 4) == 0, 'feature dimension must be multiple of 4 for sincos emb'
    omega = torch.arange(dim // 4, device = device) / (dim // 4 - 1) # shape=1024/4=256
    omega = 1. / (temperature ** omega)

    print('y.shape',y.shape) # [8,8]
    print('x.shape',x.shape) # [8,8]
    y = y.flatten()[:, None] * omega[None, :]
    x = x.flatten()[:, None] * omega[None, :] 
    print('omega.shape',omega.shape) # [256]
    print('y.shape',y.shape) # [64, 256]
    print('x.shape',x.shape) # [64, 256]

    pe = torch.cat((x.sin(), x.cos(), y.sin(), y.cos()), dim = 1)
    return pe.type(dtype)

## 前向计算模块定义，包括两个全连接层
class FeedForward(nn.Module):
    def __init__(self, dim, hidden_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.LayerNorm(dim),
            nn.Linear(dim, hidden_dim),
            nn.GELU(),
            nn.Linear(hidden_dim, dim),
        )
    def forward(self, x):
        return self.net(x)

## 注意力模块定义
class Attention(nn.Module):
    def __init__(self, dim, heads = 8, dim_head = 64):
        super().__init__()
        inner_dim = dim_head *  heads
        self.heads = heads
        self.scale = dim_head ** -0.5
        self.norm = nn.LayerNorm(dim)

        self.attend = nn.Softmax(dim = -1)

        self.to_qkv = nn.Linear(dim, inner_dim * 3, bias = False)
        self.to_out = nn.Linear(inner_dim, dim, bias = False)

    def forward(self, x):
        x = self.norm(x)

        qkv = self.to_qkv(x).chunk(3, dim = -1)
        q, k, v = map(lambda t: rearrange(t, 'b n (h d) -> b h n d', h = self.heads), qkv)

        dots = torch.matmul(q, k.transpose(-1, -2)) * self.scale

        attn = self.attend(dots)

        out = torch.matmul(attn, v)
        out = rearrange(out, 'b h n d -> b n (h d)')
        return self.to_out(out)

## Transformer模型定义
class Transformer(nn.Module):
    def __init__(self, dim, depth, heads, dim_head, mlp_dim):
        super().__init__()
        self.layers = nn.ModuleList([])
        for _ in range(depth):
            self.layers.append(nn.ModuleList([
                Attention(dim, heads = heads, dim_head = dim_head),
                FeedForward(dim, mlp_dim)
            ]))
    def forward(self, x):
        for attn, ff in self.layers:
            x = attn(x) + x
            x = ff(x) + x
        return x

## SimpleViT模型定义
class SimpleViT(nn.Module):
    def __init__(self, *, image_size, patch_size, num_classes, dim, depth, heads, mlp_dim, channels = 3, dim_head = 64):
        super().__init__()
        image_height, image_width = pair(image_size)
        patch_height, patch_width = pair(patch_size)

        assert image_height % patch_height == 0 and image_width % patch_width == 0, 'Image dimensions must be divisible by the patch size.'

        num_patches = (image_height // patch_height) * (image_width // patch_width)
        patch_dim = channels * patch_height * patch_width

        self.to_patch_embedding = nn.Sequential(
            Rearrange('b c (h p1) (w p2) -> b h w (p1 p2 c)', p1 = patch_height, p2 = patch_width),
            nn.Linear(patch_dim, dim),
        )

        self.transformer = Transformer(dim, depth, heads, dim_head, mlp_dim)

        self.to_latent = nn.Identity()
        self.linear_head = nn.Sequential(
            nn.LayerNorm(dim),
            nn.Linear(dim, num_classes)
        )

    def forward(self, img):
        *_, h, w, dtype = *img.shape, img.dtype

        x = self.to_patch_embedding(img)
        pe = posemb_sincos_2d(x)

        print('x shape',x.shape) # [1,8,8,1024]
        print('pe shape',pe.shape) # [64,1024]

        x = rearrange(x, 'b ... d -> b (...) d') + pe

        x = self.transformer(x)
        x = x.mean(dim = 1)

        x = self.to_latent(x)
        return self.linear_head(x)

model = SimpleViT(
    image_size = 256,
    patch_size = 32,
    num_classes = 1000,
    dim = 1024, ## token维度
    depth = 6, ## 模块数量
    heads = 16, ## 头的数量
    mlp_dim = 2048 ## mlp隐藏层维度
)

if __name__ == '__main__':
    img = torch.randn(1, 3, 256, 256)
    preds = model(img)
    print(preds.shape)

    torch.onnx.export(model, img, 'Simple_ViT.onnx')


```
## vit

```python
#coding:utf8
import torch
from torch import nn

from einops import rearrange, repeat
from einops.layers.torch import Rearrange

def pair(t):
    return t if isinstance(t, tuple) else (t, t)

## 预标准化方法
class PreNorm(nn.Module):
    def __init__(self, dim, fn):
        super().__init__()
        self.norm = nn.LayerNorm(dim)
        self.fn = fn
    def forward(self, x, **kwargs):
        return self.fn(self.norm(x), **kwargs)

## 前向计算模块定义，包括两个全连接层
class FeedForward(nn.Module):
    def __init__(self, dim, hidden_dim, dropout = 0.):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(dim, hidden_dim),
            nn.GELU(),
            nn.Dropout(dropout),
            nn.Linear(hidden_dim, dim),
            nn.Dropout(dropout)
        )
    def forward(self, x):
        return self.net(x)

## 注意力模块定义
class Attention(nn.Module):
    def __init__(self, dim, heads = 8, dim_head = 64, dropout = 0.):
        super().__init__()
        inner_dim = dim_head *  heads ## 8*64=512
        project_out = not (heads == 1 and dim_head == dim)

        self.heads = heads
        self.scale = dim_head ** -0.5 ## 1/sqrt(64)=1/8

        self.attend = nn.Softmax(dim = -1)
        self.dropout = nn.Dropout(dropout)

        self.to_qkv = nn.Linear(dim, inner_dim * 3, bias = False) ## 默认dim=1024，inner_dim * 3 = 512*3

        self.to_out = nn.Sequential(
            nn.Linear(inner_dim, dim),
            nn.Dropout(dropout)
        ) if project_out else nn.Identity()

    def forward(self, x):
        qkv = self.to_qkv(x).chunk(3, dim = -1) ## 从输入x，生成q,k,v，每一个维度 = inner_dim = dim_head * heads = 512
        q, k, v = map(lambda t: rearrange(t, 'b n (h d) -> b h n d', h = self.heads), qkv)

        dots = torch.matmul(q, k.transpose(-1, -2)) * self.scale ## q*k/sqrt(d)
        attn = self.attend(dots) ## 得到softmax(q*k/sqrt(d))
        attn = self.dropout(attn)

        out = torch.matmul(attn, v) ## 得到softmax(q*k/sqrt(d))*v
        out = rearrange(out, 'b h n d -> b n (h d)')
        return self.to_out(out)

## Transformer模型定义
class Transformer(nn.Module):
    def __init__(self, dim, depth, heads, dim_head, mlp_dim, dropout = 0.):
        super().__init__()
        self.layers = nn.ModuleList([])
        for _ in range(depth):
            self.layers.append(nn.ModuleList([
                PreNorm(dim, Attention(dim, heads = heads, dim_head = dim_head, dropout = dropout)),
                PreNorm(dim, FeedForward(dim, mlp_dim, dropout = dropout))
            ]))
    def forward(self, x):
        for attn, ff in self.layers:
            x = attn(x) + x ## 自注意力模块
            x = ff(x) + x ## feedforward模块
        return x

## ViT模型定义
class ViT(nn.Module):
    def __init__(self, *, image_size, patch_size, num_classes, dim, depth, heads, mlp_dim, pool = 'cls', channels = 3, dim_head = 64, dropout = 0., emb_dropout = 0.):
        super().__init__()
        image_height, image_width = pair(image_size) ## 图像尺寸
        patch_height, patch_width = pair(patch_size) ## 裁剪子图尺寸

        assert image_height % patch_height == 0 and image_width % patch_width == 0, 'Image dimensions must be divisible by the patch size.'

        num_patches = (image_height // patch_height) * (image_width // patch_width) ## 子图数量
        patch_dim = channels * patch_height * patch_width ## 展平后子图维度
        assert pool in {'cls', 'mean'}, 'pool type must be either cls (cls token) or mean (mean pooling)'

        ## 把图片切分为patch，然后拉成序列，假设输入图片大小是256x256（b,3,256,256），打算分成64个patch，每个patch是32x32像素，则rearrange操作是先变成(b,3,8x32,8x32)，最后变成(b,8x8,32x32x3)即(b,64,3072)
        self.to_patch_embedding = nn.Sequential(
            Rearrange('b c (h p1) (w p2) -> b (h w) (p1 p2 c)', p1 = patch_height, p2 = patch_width),
            nn.Linear(patch_dim, dim), ## 把子图维度映射到特定维度dim，比如32*32*3 -> 1024
        )

        self.pos_embedding = nn.Parameter(torch.randn(1, num_patches + 1, dim)) ## num_patches=64，dim=1024,+1是因为多了一个cls开启解码标志
        self.cls_token = nn.Parameter(torch.randn(1, 1, dim)) ## 额外的分类token
        self.dropout = nn.Dropout(emb_dropout)

        self.transformer = Transformer(dim, depth, heads, dim_head, mlp_dim, dropout)

        self.pool = pool
        self.to_latent = nn.Identity()

        ## 在编码器后接fc分类器head即可
        self.mlp_head = nn.Sequential(
            nn.LayerNorm(dim),
            nn.Linear(dim, num_classes)
        )

    def forward(self, img):
        x = self.to_patch_embedding(img)
        print('x shape', x.shape)
        b, n, _ = x.shape
        cls_tokens = repeat(self.cls_token, '1 1 d -> b 1 d', b = b)
        x = torch.cat((cls_tokens, x), dim=1) ## 在输入token数量维度进行拼接,## 额外追加token，变成b,65,1024
        x += self.pos_embedding[:, :(n + 1)]
        x = self.dropout(x)

        x = self.transformer(x)

        x = x.mean(dim = 1) if self.pool == 'mean' else x[:, 0] ## 取第0个token的特征，或者所有特征的平均值

        x = self.to_latent(x)
        return self.mlp_head(x)


model = ViT(
    image_size = 256,
    patch_size = 32,
    num_classes = 1000,
    dim = 1024, ## token维度
    depth = 6, ## 模块数量
    heads = 16, ## 头的数量
    mlp_dim = 2048 ## mlp隐藏层维度
)

if __name__ == '__main__':
    img = torch.randn(1, 3, 256, 256)
    preds = model(img)
    print(preds.shape)

    torch.onnx.export(model, img, 'ViT.onnx')


```

