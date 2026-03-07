---
title: 从零搭建GoogLeNet，ResNet18，ResNet50，vgg、mobilenetv1、mobilenetv2、shufflenetv1、shufflenetv2模型（Pytorch代码示例）
date: 2026-01-04 08:02:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](从零搭建深度学习模型（Pytorch代码示例一）)
## GoogLeNet
GoogLeNet 是由 Google 团队提出的一种深度卷积神经网络架构，主要特点包括：

 - Inception 模块：这是 GoogLeNet 的核心组成部分，通过在一个层内并行使用不同大小的卷积核（1x1, 3x3, 5x5）和池化操作，然后将它们的结果拼接在一起，形成一个更丰富的特征表示。
 - 辅助分类器：为了缓解梯度消失问题，GoogLeNet 在网络的中间层添加了两个辅助分类器，这些分类器在训练过程中提供额外的梯度信号，并且有助于正则化模型。
 - 全局平均池化：在网络的最后，GoogLeNet 使用全局平均池化替代全连接层，这不仅减少了参数量，还提高了模型的泛化能力。
```python
#coding:utf8

# Copyright 2023 longpeng2008. All Rights Reserved.
# Licensed under the Apache License, Version 2.0 (the "License");
# If you find any problem,please contact us
#
#     longpeng2008to2012@gmail.com
#
# or create issues
# =============================================================================
import torch
import torch.nn as nn

# 卷积模块
class BasicConv(nn.Module):
    def __init__(self,in_channels,out_channels,kernel_size,stride=(1,1),padding=(0,0)):
        super(BasicConv, self).__init__()
        self.conv = nn.Conv2d(in_channels=in_channels,out_channels=out_channels,kernel_size=kernel_size,stride=stride,padding=padding)
        self.relu = nn.ReLU(inplace=True)

    def forward(self, x):
        x = self.conv(x)
        x = self.relu(x)
        return x

# 额外的损失分支
class SideBranch(nn.Module):
    def __init__(self, in_channels,num_classes):
        super(SideBranch, self).__init__()
        self.avg_pool = nn.AvgPool2d(kernel_size=5, stride=3)
        self.conv1x1 = BasicConv(in_channels=in_channels, out_channels=128, kernel_size=1)
        self.fc_1 = nn.Linear(in_features=2048, out_features=1024)
        self.relu = nn.ReLU(inplace=True)
        self.fc_2 = nn.Linear(in_features=1024, out_features=num_classes)

    def forward(self, x):
        x = self.avg_pool(x)
        x = self.conv1x1(x)
        x = torch.flatten(x,1)
        x = self.fc_1(x)
        x = self.relu(x)
        x = torch.dropout(x, 0.7, train=True)
        x = self.fc_2(x)
        return x

# Inception模块
class InceptionBlock(nn.Module):
    def __init__(self,in_channels,ch1x1, ch3x3reduce,ch3x3,ch5x5reduce,ch5x5,chpool):
        super(InceptionBlock, self).__init__()
        self.branch_1 = BasicConv(in_channels=in_channels, out_channels=ch1x1,kernel_size=1)
        self.branch_2 = nn.Sequential(
            BasicConv(in_channels=in_channels, out_channels=ch3x3reduce, kernel_size=1),
            BasicConv(in_channels=ch3x3reduce, out_channels=ch3x3,kernel_size=3, padding=1)
        )
        self.branch_3 = nn.Sequential(
            BasicConv(in_channels=in_channels, out_channels=ch5x5reduce,kernel_size=1),
            BasicConv(in_channels=ch5x5reduce,out_channels=ch5x5,kernel_size=5, padding=2)
        )
        self.branch_4 = nn.Sequential(
            nn.MaxPool2d(kernel_size=3, padding=1,stride=1, ceil_mode=True),
            BasicConv(in_channels=in_channels,out_channels=chpool,kernel_size=1)
        )

    def forward(self, x):
        x_1 = self.branch_1(x)
        x_2 = self.branch_2(x)
        x_3 = self.branch_3(x)
        x_4 = self.branch_4(x)
        x = torch.cat([x_1, x_2, x_3, x_4], dim=1)
        return x

# GoogLeNet/Inception模型
class Inception_V1(nn.Module):
    def __init__(self, num_classes):
        super(Inception_V1, self).__init__()
        self.BasicConv_1 = BasicConv(in_channels=3, out_channels=64, kernel_size=7,stride=2, padding=3)
        self.max_pool_1 = nn.MaxPool2d(kernel_size=3, stride=2, ceil_mode=True)   # 把不足square_size的边保留下来，单独计算
        self.lrn_1 = nn.LocalResponseNorm(2)

        self.conv_1x1 = BasicConv(in_channels=64, out_channels=64, kernel_size=1)
        self.conv_3x3 = BasicConv(in_channels=64, out_channels=192, kernel_size=3, padding=1)
        self.lrn_2 = nn.LocalResponseNorm(2)
        self.max_pool_2 = nn.MaxPool2d(kernel_size=3, stride=2, ceil_mode=True)

        #   in_channels,ch1x1, ch3x3reduce,ch3x3,ch5x5reduce,ch5x5,chpool
        self.InceptionBlock_3a = InceptionBlock(192,64,96,128,16,32,32)
        self.InceptionBlock_3b = InceptionBlock(256,128,128,192,32,96,64)
        self.max_pool_3 = nn.MaxPool2d(kernel_size=3, stride=2, ceil_mode=True)

        self.InceptionBlock_4a = InceptionBlock(480,192,96,208,16,48,64)

        self.SideBranch_1 = SideBranch(512, num_classes)

        self.InceptionBlock_4b = InceptionBlock(512,160,112,224,24,64,64)
        self.InceptionBlock_4c = InceptionBlock(512,128,128,256,24,64,64)
        self.InceptionBlock_4d = InceptionBlock(512,112,144,288,32,64,64)

        self.SideBranch_2 = SideBranch(528, num_classes)

        self.InceptionBlock_4e = InceptionBlock(528,256,160,320,32,128,128)

        self.max_pool_4 = nn.MaxPool2d(kernel_size=3,stride=2, ceil_mode=True)

        self.InceptionBlock_5a = InceptionBlock(832,256,160,320,32,128,128)
        self.InceptionBlock_5b = InceptionBlock(832,384,192,384,48,128,128)

        self.avg_pool = nn.AdaptiveAvgPool2d(1)
        self.flatten = nn.Flatten()
        self.fc = nn.Linear(in_features=1024 ,out_features=num_classes)

    def forward(self, x):
        x = self.BasicConv_1(x)
        x = self.max_pool_1(x)
        x = self.lrn_1(x)

        x = self.conv_1x1(x)
        x = self.conv_3x3(x)
        x = self.lrn_1(x)
        x = self.max_pool_2(x)

        x = self.InceptionBlock_3a(x)
        x = self.InceptionBlock_3b(x)
        x = self.max_pool_3(x)

        x = self.InceptionBlock_4a(x)

        x_1 = self.SideBranch_1(x)

        x = self.InceptionBlock_4b(x)
        x = self.InceptionBlock_4c(x)
        x = self.InceptionBlock_4d(x)

        x_2 = self.SideBranch_2(x)

        x = self.InceptionBlock_4e(x)

        x = self.max_pool_4(x)

        x = self.InceptionBlock_5a(x)
        x = self.InceptionBlock_5b(x)

        x = self.avg_pool(x)
        x = self.flatten(x)
        x = torch.dropout(x, 0.4,train=True)
        x = self.fc(x)

        x_1 = torch.softmax(x_1, dim=1)
        x_2 = torch.softmax(x_2, dim=1)
        x_3 = torch.softmax(x, dim=1)

        # output = x_3 + (x_1 + x_2) * 0.3
        return x_3,x_2,x_1


if __name__ == '__main__':
    # 创建模型，给定输入，前向传播，存储模型
    input = torch.randn([1, 3, 224, 224])
    model = Inception_V1(num_classes=1000)
    torch.save(model, 'googlenet.pth')

    x_3,x_2,x_1 = model(input)

    # 观察输出，只需要观察shape是我们想要的即可
    print(x_1.shape)
    print(x_2.shape)
    print(x_3.shape)

    torch.onnx.export(model, input, 'googlenet.onnx')

```

## ResNet18
ResNet18 是 ResNet（残差网络）系列中的一个较小型号，由 Microsoft Research 提出。ResNet 通过引入残差块（Residual Block）解决了深度神经网络中的梯度消失问题，使得网络可以更深，从而提高模型的性能。以下是 ResNet18 的主要特点和结构：
### 主要特点

 - 残差块：每个残差块包含两个卷积层，每个卷积层后面跟着一个批量归一化（Batch Normalization）层和一个 ReLU
   激活函数。残差块通过跳跃连接（Skip Connection）将输入直接加到输出上，这样可以缓解梯度消失问题。
 - 简单的结构：ResNet18 只有 18 层，相对于其他更深的 ResNet 模型（如 ResNet50、ResNet101）来说，计算量较小，适合资源有限的场景。

### 结构
ResNet18 的结构可以分为以下几个部分：
输入层：

 - 卷积层：7x7 卷积核，64 个输出通道，步长为 2，填充为 3。
 - 批量归一化层。
 - ReLU 激活函数。
 - 最大池化层：3x3 核，步长为 2，填充为 1。

残差块：

 - 4 个残差块组，每个组包含 2 个残差块。
 - 每个残差块包含两个 3x3 卷积层，每个卷积层后面跟着一个批量归一化层和一个 ReLU 激活函数。
 - 跳跃连接将输入直接加到输出上。

全局平均池化层：

 - 将特征图缩小到固定大小，通常为 1x1。 

全连接层：

 - 输出分类结果，通常用于图像分类任务。

```python
#coding:utf8

# Copyright 2023 longpeng2008. All Rights Reserved.
# Licensed under the Apache License, Version 2.0 (the "License");
# If you find any problem,please contact us
#
#     longpeng2008to2012@gmail.com
#
# or create issues
# =============================================================================
import torch
import torch.nn as nn

# 残差模块
class ResidualBlock(nn.Module):
    def __init__(self, in_channels, outchannels):
        super(ResidualBlock, self).__init__()
        self.channel_equal_flag = True
        if in_channels == outchannels:
            self.conv1 = nn.Conv2d(in_channels=in_channels, out_channels=outchannels, kernel_size=3, padding=1, stride=1)
        else:
            ## 对恒等映射分支的变换，当通道数发生变换时，分辨率变为原来的二分之一
            self.conv1x1 = nn.Conv2d(in_channels=in_channels, out_channels=outchannels, kernel_size=1, stride=2)
            self.bn1x1 = nn.BatchNorm2d(num_features=outchannels)
            self.channel_equal_flag = False

            self.conv1 = nn.Conv2d(in_channels=in_channels, out_channels=outchannels,kernel_size=3,padding=1, stride=2)

        self.bn1 = nn.BatchNorm2d(num_features=outchannels)
        self.relu = nn.ReLU(inplace=True)

        self.conv2 = nn.Conv2d(in_channels=outchannels, out_channels=outchannels, kernel_size=3,padding=1)
        self.bn2 = nn.BatchNorm2d(num_features=outchannels)

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

        out = identity + x
        return out

class ResNet18(nn.Module):
    def __init__(self, num_classes=1000):
        super(ResNet18, self).__init__()
        # conv1
        self.conv1 = nn.Conv2d(in_channels=3, out_channels=64,kernel_size=7,stride=2, padding=3)
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

        # softmax
        self.softmax = nn.Softmax(dim=1)

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

        # avgpool + fc + softmax
        x = self.avg_pool(x)
        x = torch.flatten(x, 1)
        x = self.fc(x)
        x = self.softmax(x)

        return x

if __name__ == '__main__':
    model = ResNet18(num_classes=1000)
    input = torch.randn([1, 3, 224, 224])
    output = model(input)

    torch.save(model, 'resnet18.pth')
    torch.onnx.export(model,input,'resnet18.onnx')

```
### 示例

```python
import torch
from torch import nn
from torch.utils.data import DataLoader
from tqdm import tqdm  # pip install tqdm
import matplotlib.pyplot as plt
import os
from torchsummary import summary

from torch.utils.tensorboard import SummaryWriter

import wandb
import datetime

from MyModel import SimpleCNN

from torchvision.models import resnet18, ResNet18_Weights

from MYDataset import MyDataset

# # 定义训练函数
def train(dataloader, model, loss_fn, optimizer):
    # 初始化训练数据集的大小和批次数量
    size = len(dataloader.dataset)
    num_batches = len(dataloader)
    # 设置模型为训练模式
    model.train()
    # 初始化总损失和正确预测数量
    loss_total = 0
    correct = 0
    # 遍历数据加载器中的所有数据批次
    for X, y in tqdm(dataloader):
        # 将数据和标签移动到指定设备（例如GPU）
        X, y = X.to(device), y.to(device)
        # 使用模型进行预测
        pred = model(X)
        # 计算正确预测的数量
        correct += (pred.argmax(1) == y).type(torch.float).sum().item()
        # 计算预测结果和真实结果之间的损失
        loss = loss_fn(pred, y)
        # 累加总损失
        loss_total += loss.item()
        # 执行反向传播，计算梯度
        loss.backward()
        # 更新模型参数
        optimizer.step()
        # 清除梯度信息
        optimizer.zero_grad()

    # 计算平均损失和准确率
    loss_avg = loss_total / num_batches
    correct /= size
    # 返回准确率和平均损失，保留三位小数
    return round(correct, 3), round(loss_avg,3)

# 定义测试函数
def test(dataloader, model, loss_fn):
    # 初始化测试数据集的大小和批次数量
    size = len(dataloader.dataset)
    num_batches = len(dataloader)
    # 设置模型为评估模式
    model.eval()

    # 初始化测试损失和正确预测数量
    test_loss, correct = 0, 0

    # 不计算梯度，以提高计算效率并减少内存使用
    with torch.no_grad():
        # 遍历数据加载器中的所有数据批次
        for X, y in tqdm(dataloader):
            # 将数据和标签移动到指定设备（例如GPU）
            X, y = X.to(device), y.to(device)
            # 使用模型进行预测
            pred = model(X)
            # 累加预测损失
            test_loss += loss_fn(pred, y).item()
            # 累加正确预测的数量
            correct += (pred.argmax(1) == y).type(torch.float).sum().item()

    # 计算平均测试损失和准确率
    test_loss /= num_batches
    correct /= size

    # 返回准确率和平均测试损失，保留三位小数
    return round(correct, 3), round(test_loss, 3)

def writedata(txt_log_name, tensorboard_writer, epoch, train_accuracy, train_loss, test_accuracy, test_loss):
    # 保存到文档
    with open(txt_log_name, "a+") as f:
        f.write(f"Epoch:{epoch}\ttrain_accuracy:{train_accuracy}\ttrain_loss:{train_loss}\ttest_accuracy:{test_accuracy}\ttest_loss:{test_loss}\n")

    # 保存到tensorboard
    # 记录全连接层参数
    for name, param in model.named_parameters():
        tensorboard_writer.add_histogram(name, param.clone().cpu().data.numpy(), global_step=epoch)

    tensorboard_writer.add_scalar('Accuracy/train', train_accuracy, epoch)
    tensorboard_writer.add_scalar('Loss/train', train_loss, epoch)
    tensorboard_writer.add_scalar('Accuracy/test', test_accuracy, epoch)
    tensorboard_writer.add_scalar('Loss/test', test_loss, epoch)

    wandb.log({"Accuracy/train": train_accuracy,
               "Loss/train": train_loss,
               "Accuracy/test": test_accuracy,
               "Loss/test": test_loss})

def plot_txt(log_txt_loc):
    with open(log_txt_loc, 'r') as f:
        log_data = f.read()

    # 解析日志数据
    epochs = []
    train_accuracies = []
    train_losses = []
    test_accuracies = []
    test_losses = []

    for line in log_data.strip().split('\n'):
        epoch, train_acc, train_loss, test_acc, test_loss = line.split('\t')
        epochs.append(int(epoch.split(':')[1]))
        train_accuracies.append(float(train_acc.split(':')[1]))
        train_losses.append(float(train_loss.split(':')[1]))
        test_accuracies.append(float(test_acc.split(':')[1]))
        test_losses.append(float(test_loss.split(':')[1]))

    # 创建折线图
    plt.figure(figsize=(10, 5))

    # 训练数据
    plt.subplot(1, 2, 1)
    plt.plot(epochs, train_accuracies, label='Train Accuracy')
    plt.plot(epochs, test_accuracies, label='Test Accuracy')
    plt.title('Training Metrics')
    plt.xlabel('Epoch')
    plt.ylabel('Value')
    plt.legend()
    # 设置横坐标刻度为整数
    plt.xticks(range(min(epochs), max(epochs) + 1))

    # 测试数据
    plt.subplot(1, 2, 2)
    plt.plot(epochs, train_losses, label='Train Loss')
    plt.plot(epochs, test_losses, label='Test Loss')
    plt.title('Testing Metrics')
    plt.xlabel('Epoch')
    plt.ylabel('Value')
    plt.legend()
    # 设置横坐标刻度为整数
    plt.xticks(range(min(epochs), max(epochs) + 1))

    plt.tight_layout()
    plt.show()



if __name__ == '__main__':
    batch_size = 64
    init_lr = 1e-4
    epochs = 20
    log_root = "logs_resnet18_adam"
    log_txt_loc = os.path.join(log_root,"log.txt")

    # 指定TensorBoard数据的保存地址
    tensorboard_writer = SummaryWriter(log_root)

    # WandB信息保存地址
    run_time = datetime.datetime.now().strftime("%Y-%m-%d-%H-%M-%S")
    wandb.init(
        dir=log_root,
        project='Flower Classify',
        name=f"run_resnet18_adam",
        config={
            "learning_rate": init_lr,
            "batch_size": batch_size,
            "model": "resnet18",
            "dataset": "Flower10",
            "epochs": epochs,
        }
    )

    if os.path.isdir(log_root):
        pass
    else:
        os.mkdir(log_root)

    train_data = MyDataset("train.txt",train_flag=True)
    test_data = MyDataset("test.txt",train_flag=False)

    # 创建数据加载器
    train_dataloader = DataLoader(train_data, batch_size=batch_size)
    test_dataloader = DataLoader(test_data, batch_size=batch_size)

    for X, y in test_dataloader:
        print(f"Shape of X [N, C, H, W]: {X.shape}")
        print(f"Shape of y: {y.shape} {y.dtype}")
        break

    # 指定设备
    device = "cuda" if torch.cuda.is_available() else "cpu"

    print(f"Using {device} device")


    """
        使用resnet18进行预训练
    """

    # 加载具有预训练权重的ResNet18模型
    weights = ResNet18_Weights.DEFAULT
    model = resnet18(weights=weights)
    # 保存模型权重到本地文件
    torch.save(model.state_dict(), 'resnet18_weights.pth')
    print("ResNet18 weights downloaded and saved as 'resnet18_weights.pth'")

    pretrain_model = resnet18(pretrained=False)
    print(resnet18)

    num_ftrs = pretrain_model.fc.in_features    # 获取全连接层的输入
    pretrain_model.fc = nn.Linear(num_ftrs, 10)  # 全连接层改为不同的输出

    # 预先训练好的参数， 'https://download.pytorch.org/models/resnet34-333f7ec4.pth'
    pretrained_dict = torch.load('resnet18_weights.pth')

    # # 弹出fc层的参数
    pretrained_dict.pop('fc.weight')
    pretrained_dict.pop('fc.bias')

    # # 自己的模型参数变量，在开始时里面参数处于初始状态，所以很多0和1
    model_dict = pretrain_model.state_dict()

    # # 去除一些不需要的参数
    pretrained_dict = {k: v for k, v in pretrained_dict.items() if k in model_dict}

    # # 模型参数列表进行参数更新，加载参数
    model_dict.update(pretrained_dict)

    # 改进过的预训练模型结构，加载刚刚的模型参数列表
    pretrain_model.load_state_dict(model_dict)

    '''
        冻结部分层
    '''
    # 将满足条件的参数的 requires_grad 属性设置为False
    for name, value in pretrain_model.named_parameters():
        if (name != 'fc.weight') and (name != 'fc.bias'):
            value.requires_grad = False
    #
    # filter 函数将模型中属性 requires_grad = True 的参数选出来
    params_selected = filter(lambda p: p.requires_grad, pretrain_model.parameters())    # 要更新的参数在 params_selected 当中


    model = pretrain_model.to(device)



    print(model)
    summary(model, (3,224,224))

    # 模拟输入，大小和输入相同即可
    init_img = torch.zeros((1, 3, 224, 224), device=device)
    tensorboard_writer.add_graph(model, init_img)

    # 添加wandb的模型记录
    wandb.watch(model, log='all', log_graph=True)

    # 定义损失函数
    loss_fn = nn.CrossEntropyLoss()
    # 定义优化器
    optimizer = torch.optim.Adam(params_selected, lr=init_lr)

    best_acc = 0
    # 定义循环次数，每次循环里面，先训练，再测试
    for t in range(epochs):
        print(f"Epoch {t + 1}\n-------------------------------")
        train_acc, train_loss = train(train_dataloader, model, loss_fn, optimizer)
        test_acc, test_loss = test(test_dataloader, model, loss_fn)
        writedata(log_txt_loc, tensorboard_writer,t,train_acc,train_loss,test_acc,test_loss)

        # 保存最佳模型
        if test_acc > best_acc:
            best_acc = test_acc
            torch.save(model.state_dict(), os.path.join(log_root,"best.pth"))

        torch.save(model.state_dict(), os.path.join(log_root,"last.pth"))

    print("Done!")

    tensorboard_writer.close()
    wandb.finish()
    plot_txt(log_txt_loc)

```

## ResNet50

```python
# coding:utf8

# Copyright 2023 longpeng2008. All Rights Reserved.
# Licensed under the Apache License, Version 2.0 (the "License");
# If you find any problem,please contact us
#
#     longpeng2008to2012@gmail.com
#
# or create issues
# =============================================================================
import torch
import torch.nn as nn

class ResidualBlock(nn.Module):
    def __init__(self, in_channels, out_channels, identity_downsample=None, stride=1):
        super(ResidualBlock, self).__init__()

        self.expansion = 4

        self.conv1 = nn.Conv2d(in_channels=in_channels, out_channels=out_channels, kernel_size=1, stride=1, padding=0)
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.conv2 = nn.Conv2d(in_channels=out_channels, out_channels=out_channels, kernel_size=3, stride=stride,
                               padding=1)
        self.bn2 = nn.BatchNorm2d(out_channels)
        self.conv3 = nn.Conv2d(in_channels=out_channels, out_channels=out_channels * self.expansion, kernel_size=1,
                               stride=1, padding=0)
        self.bn3 = nn.BatchNorm2d(out_channels * self.expansion)
        self.relu = nn.ReLU()
        self.identity_downsample = identity_downsample
        self.stride = stride

    def forward(self, x):
        identity = x.clone()

        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)
        x = self.conv2(x)
        x = self.bn2(x)
        x = self.relu(x)
        x = self.conv3(x)
        x = self.bn3(x)

        if self.identity_downsample is not None:
            identity = self.identity_downsample(identity)

        x += identity
        x = self.relu(x)

        return x


class ResNet(nn.Module):
    def __init__(self, block, layers, num_classes):
        super(ResNet, self).__init__()

        self.in_channels = 64

        self.conv1 = nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3)
        self.bn1 = nn.BatchNorm2d(64)
        self.relu = nn.ReLU()
        self.pooling = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)

        self.layer1 = self.res_layer(block, layers[0], out_channels=64, stride=1) #第1组残差模块，整体分辨率不下降
        self.layer2 = self.res_layer(block, layers[1], out_channels=128, stride=2)#第2组残差模块，分辨率下降1/2
        self.layer3 = self.res_layer(block, layers[2], out_channels=256, stride=2)#第2组残差模块，分辨率下降1/2
        self.layer4 = self.res_layer(block, layers[3], out_channels=512, stride=2)#第2组残差模块，分辨率下降1/2

        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512 * 4, num_classes)

    def forward(self, x):
        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)
        x = self.pooling(x)
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)
        x = self.avgpool(x)

        x = x.reshape(x.shape[0], -1)

        x = self.fc(x)

        return x

    def res_layer(self, block, num_residual_blocks, out_channels, stride):
        identity_downsample = None
        layers = []

        # 每一组残差模块的第一个残差模块需要特殊对待
        if stride != 1 or self.in_channels != out_channels * 4:
            identity_downsample = nn.Sequential(
                nn.Conv2d(
                    self.in_channels,
                    out_channels * 4,
                    kernel_size=1,
                    stride=stride,
                ),
                nn.BatchNorm2d(out_channels * 4),
            )

        layers.append(
            block(self.in_channels, out_channels, identity_downsample, stride)
        )

        # 除了每一组残差模块的第一个，其他残差模块的输入通道都等于输出通道的4倍，ResNet 50,101,152
        self.in_channels = out_channels * 4 # 该值每经过一组残差模块，就会变大，64 -> 4*64=256 -> 4*128=512 -> 4*256=1024 -> 4*512=2048
        # 如resnet50的conv2_x，通道变换为256 -> 64 -> 256

        for i in range(num_residual_blocks - 1):
            layers.append(block(self.in_channels, out_channels))

        return nn.Sequential(*layers)


def ResNet50(num_classes=1000):
    return ResNet(ResidualBlock, [3, 4, 6, 3], num_classes)


def ResNet101(num_classes=1000):
    return ResNet(ResidualBlock, [3, 4, 23, 3], num_classes)


def ResNet152(num_classes=1000):
    return ResNet(ResidualBlock, [3, 8, 36, 3], num_classes)


input = torch.randn([1, 3, 224, 224])
resnet50 = ResNet50(num_classes=1000)
output = resnet50(input)

torch.save(resnet50, 'resnet50.pth')
torch.onnx.export(resnet50, input, 'resnet50.onnx')


```

## vgg
VGG（Visual Geometry Group）是由牛津大学的 Visual Geometry Group 提出的一种经典的卷积神经网络架构。VGG 网络在 2014 年的 ImageNet 大规模视觉识别挑战赛（ILSVRC）中取得了优异的成绩，以其简单而有效的设计而闻名。
### 主要特点

 - 简单且一致的架构：VGG 使用了非常简单的架构，主要由多个 3x3 的卷积层和最大池化层组成。
 - 多层卷积：通过堆叠多个小卷积层（3x3）来增加网络的深度，而不是使用更大的卷积核。
 - 全连接层：在网络的最后部分，使用了几个全连接层来进行分类。

### 结构
VGG 网络有多种变体，其中最常用的是 VGG16 和 VGG19。以下是 VGG16 的结构：
输入层：

 - 输入图像尺寸：224x224x3。

卷积层：

 - 2 个 3x3 卷积层，64 个输出通道，后接一个 2x2 最大池化层。 
 - 2 个 3x3 卷积层，128 个输出通道，后接一个 2x2 最大池化层。 
 - 3 个 3x3 卷积层，256 个输出通道，后接一个 2x2 最大池化层。 
 - 3 个 3x3 卷积层，512 个输出通道，后接一个 2x2 最大池化层。 
 - 3 个 3x3 卷积层，512 个输出通道，后接一个 2x2 最大池化层。

全连接层：

 - 2 个 4096 维的全连接层，每个全连接层后面跟着一个 ReLU 激活函数和一个 Dropout 层（防止过拟合）。
 - 1 个 1000 维的全连接层，用于输出分类结果。

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import numpy as np

class VGG(nn.Module):
    def __init__(self):
        super(VGG,self).__init__()
        self.conv1_1 = nn.Conv2d(3,64,3,1,1)
        self.conv1_2 = nn.Conv2d(64,64,3,1,1)
        self.pool1 = nn.MaxPool2d(2)

        self.conv2_1 = nn.Conv2d(64,128,3,1,1)
        self.conv2_2 = nn.Conv2d(128,128,3,1,1)
        self.pool2 = nn.MaxPool2d(2)

        self.conv3_1 = nn.Conv2d(128, 256, 3, 1, 1)
        self.conv3_2 = nn.Conv2d(256, 256, 3, 1, 1)
        self.conv3_3 = nn.Conv2d(256, 256, 3, 1, 1)
        self.pool3 = nn.MaxPool2d(2)

        self.conv4_1 = nn.Conv2d(256, 512, 3, 1, 1)
        self.conv4_2 = nn.Conv2d(512, 512, 3, 1, 1)
        self.conv4_3 = nn.Conv2d(512, 512, 3, 1, 1)
        self.pool4 = nn.MaxPool2d(2)

        self.conv5_1 = nn.Conv2d(512, 512, 3, 1, 1)
        self.conv5_2 = nn.Conv2d(512, 512, 3, 1, 1)
        self.conv5_3 = nn.Conv2d(512, 512, 3, 1, 1)
        self.pool5 = nn.MaxPool2d(2)

        self.fc1 = nn.Linear(7*7*512,4096)
        self.fc2 = nn.Linear(4096, 4096)
        self.classifier = nn.Linear(4096, 1000)

    def forward(self,x):
        x = F.relu(self.conv1_1(x))
        x = F.relu(self.conv1_2(x))
        x = self.pool1(x)

        x = F.relu(self.conv2_1(x))
        x = F.relu(self.conv2_2(x))
        x = self.pool2(x)

        x = F.relu(self.conv3_1(x))
        x = F.relu(self.conv3_2(x))
        x = F.relu(self.conv3_3(x))
        x = self.pool3(x)

        x = F.relu(self.conv4_1(x))
        x = F.relu(self.conv4_2(x))
        x = F.relu(self.conv4_3(x))
        x = self.pool4(x)

        x = F.relu(self.conv5_1(x))
        x = F.relu(self.conv5_2(x))
        x = F.relu(self.conv5_3(x))
        x = self.pool5(x)

        #print('conv5.shape',x.shape) #n*7*7*512
        x = x.reshape(-1,7*7*512)
        #print('conv5.shape',x.shape) #n*7*7*512

        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.classifier(x)

        return x

x = torch.randn((1,3,224,224))
vgg = VGG()
y = vgg(x)
print(y.shape)

torch.save(vgg,'vgg.pth')
torch.onnx.export(vgg,x,'vgg.onnx')
```
## mobilenetv1
MobileNetV1 是一种轻量级的卷积神经网络架构，主要用于移动设备和嵌入式视觉应用。以下是 MobileNetV1 的一些关键特点：

 - 深度可分离卷积：MobileNetV1 引入了深度可分离卷积，将标准卷积分解为深度卷积（depthwise
   convolution）和逐点卷积（pointwise convolution）。这种设计大大减少了模型的参数数量和计算复杂度。
   结构简单：MobileNetV1 的结构相对简单，由多个深度可分离卷积层堆叠而成，每个层后面通常跟着批量归一化（Batch Normalization）和ReLU激活函数。
   高效：由于其轻量级的设计，MobileNetV1 在资源受限的设备上表现出色，适用于实时图像分类、物体检测等任务。
### MobileNetV1 的基本结构
一个典型的 MobileNetV1 模型包括以下几个部分：
 - 输入层：接受输入图像。
 - 初始卷积层：一个标准的3x3卷积层，后面跟着批量归一化和ReLU激活。
 - 深度可分离卷积层：多个深度可分离卷积层，每个层包括深度卷积、批量归一化、ReLU激活、逐点卷积、批量归一化和ReLU激活。
 - 全局平均池化层：将特征图降维为固定大小的向量。
 - 全连接层：输出分类结果。

```python
#coding:utf8
import time 
import torch
import torch.nn as nn
import torchvision.models as models
 
class MobileNetV1(nn.Module):
    def __init__(self):
        super(MobileNetV1,self).__init__()
        
        # 标准卷积
        def conv_bn(inp,oup,stride):
            return nn.Sequential(
                    nn.Conv2d(inp,oup,3,stride,1,bias = False),
                    nn.BatchNorm2d(oup),
                    nn.ReLU(inplace = True))
        
        # 深度可分离卷积，depthwise convolution + pointwise convolution
        def conv_dw(inp,oup,stride):
            return nn.Sequential(
                    nn.Conv2d(inp,inp,3,stride,1,groups = inp,bias = False),
                    nn.BatchNorm2d(inp),
                    nn.ReLU(inplace = True),
                    
                    nn.Conv2d(inp,oup,1,1,0,bias = False),
                    nn.BatchNorm2d(oup),
                    nn.ReLU(inplace = True))
            
        #网络模型声明
        self.model = nn.Sequential(
                conv_bn(3,32,2),
                conv_dw(32,64,1),
                conv_dw(64,128,2),
                conv_dw(128,128,1),
                conv_dw(128,256,2),
                conv_dw(256,256,1),
                conv_dw(256,512,2),
                conv_dw(512,512,1),
                conv_dw(512,512,1),
                conv_dw(512,512,1),
                conv_dw(512,512,1),
                conv_dw(512,512,1),
                conv_dw(512,1024,2),
                conv_dw(1024,1024,1),
                nn.AvgPool2d(7),)
      
        self.fc = nn.Linear(1024,1000)
    
    #网络的前向过程    
    def forward(self,x):
        x = self.model(x)
        x = x.view(-1,1024)
        x = self.fc(x)
        return x

#速度评估
def speed(model,name):
    t0 = time.time()
    input = torch.rand(1,3,224,224).cpu()
    t1 = time.time()
    
    model(input)
    t2 = time.time()

    for i in range(0,30):
        model(input)
    t3 = time.time()
    
    print('%10s : %f'%(name,(t3 - t2)/30))
 
if __name__ == '__main__':
    resnet18 = models.resnet18().cpu()
    alexnet = models.alexnet().cpu()
    vgg16 = models.vgg16().cpu()
    mobilenetv1 = MobileNetV1().cpu()
    
    speed(resnet18,'resnet18')
    speed(alexnet,'alexnet')
    speed(vgg16,'vgg16')
    speed(mobilenetv1,'mobilenet')

```
## mobilenetv2
MobileNetV2 是 MobileNetV1 的改进版本，进一步优化了模型的性能和效率。MobileNetV2 引入了一些新的设计理念，使其在保持轻量级的同时，提高了模型的准确性和鲁棒性。以下是 MobileNetV2 的一些关键特点：

 - 倒残差结构（Inverted Residuals）：与传统的残差块不同，MobileNetV2 使用“膨胀-压缩”的策略，即先通过1x1卷积增加通道数，再进行深度卷积，最后通过1x1卷积减少通道数。
 - 线性瓶颈（Linear Bottlenecks）：在每个残差块的末端使用线性层，而不是ReLU激活函数，以保留更多的信息。
 - 跳跃连接（Skip Connections）：类似于ResNet中的跳跃连接，有助于梯度传播，提高训练效果。
### MobileNetV2 的基本结构
一个典型的 MobileNetV2 模型包括以下几个部分：
 - 输入层：接受输入图像。
 - 初始卷积层：一个标准的3x3卷积层，后面跟着批量归一化和ReLU6激活。
 - 倒残差块（Inverted Residual Blocks）：多个倒残差块，每个块包括1x1卷积、深度卷积、1x1卷积和跳跃连接。
 - 全局平均池化层：将特征图降维为固定大小的向量。
 - 全连接层：输出分类结果。
```python
#coding:utf8
import time
import torch
import torch.nn as nn

# 调整通道数量，使其都可以除8
def _make_divisible(v, divisor, min_value=None):
    if min_value is None:
        min_value = divisor
    new_v = max(min_value, int(v + divisor / 2) // divisor * divisor)
    # 保证向下取整损失不要超过10%.
    if new_v < 0.9 * v:
        new_v += divisor
    return new_v

# 标准卷积模块
class ConvBNReLU(nn.Sequential):
    def __init__(self, in_planes, out_planes, kernel_size=3, stride=1, groups=1):
        padding = (kernel_size - 1) // 2
        super(ConvBNReLU, self).__init__(
            nn.Conv2d(in_planes, out_planes, kernel_size, stride, padding, groups=groups, bias=False),
            nn.BatchNorm2d(out_planes),
            nn.ReLU6(inplace=True)
        )

# 反转残差模块
class InvertedResidual(nn.Module):
    def __init__(self, inp, oup, stride, expand_ratio):
        super(InvertedResidual, self).__init__()
        self.stride = stride
        assert stride in [1, 2]

        hidden_dim = int(round(inp * expand_ratio))
        self.use_res_connect = self.stride == 1 and inp == oup

        layers = []
        if expand_ratio != 1:
            # pw
            layers.append(ConvBNReLU(inp, hidden_dim, kernel_size=1))
        layers.extend([
            # dw
            ConvBNReLU(hidden_dim, hidden_dim, stride=stride, groups=hidden_dim),
            # pw-linear
            nn.Conv2d(hidden_dim, oup, 1, 1, 0, bias=False),
            nn.BatchNorm2d(oup),
        ])
        self.conv = nn.Sequential(*layers)

    def forward(self, x):
        if self.use_res_connect:
            return x + self.conv(x)
        else:
            return self.conv(x)

# 模型定义模块
class MobileNetV2(nn.Module):
    def __init__(self, num_classes=1000, width_mult=1.0, inverted_residual_setting=None, round_nearest=8):
        super(MobileNetV2, self).__init__()
        block = InvertedResidual
        input_channel = 32
        last_channel = 1280

        if inverted_residual_setting is None:
            inverted_residual_setting = [
                # t, c, n, s
                [1, 16, 1, 1],
                [6, 24, 2, 2],
                [6, 32, 3, 2],
                [6, 64, 4, 2],
                [6, 96, 3, 1],
                [6, 160, 3, 2],
                [6, 320, 1, 1],
            ]

        # 需要输入t,c,n,s参数
        if len(inverted_residual_setting) == 0 or len(inverted_residual_setting[0]) != 4:
            raise ValueError("inverted_residual_setting should be non-empty "
                             "or a 4-element list, got {}".format(inverted_residual_setting))

        # 第一层
        input_channel = _make_divisible(input_channel * width_mult, round_nearest)
        self.last_channel = _make_divisible(last_channel * max(1.0, width_mult), round_nearest)
        features = [ConvBNReLU(3, input_channel, stride=2)]
        # inverted residual blocks
        for t, c, n, s in inverted_residual_setting:
            output_channel = _make_divisible(c * width_mult, round_nearest)
            for i in range(n):
                stride = s if i == 0 else 1
                features.append(block(input_channel, output_channel, stride, expand_ratio=t))
                input_channel = output_channel
        # 最后几层
        features.append(ConvBNReLU(input_channel, self.last_channel, kernel_size=1))
        # make it nn.Sequential
        self.features = nn.Sequential(*features)

        # building classifier
        self.classifier = nn.Sequential(
            nn.Dropout(0.2),
            nn.Linear(self.last_channel, num_classes),
        )

    def forward(self, x):
        x = self.features(x) # 7*7*1280
        x = x.mean([2, 3])
        x = self.classifier(x)
        return x

# 速度评估
def speed(model, name):
    t0 = time.time()
    input = torch.rand(1, 3, 224, 224).cpu()
    t1 = time.time()

    model(input)
    t2 = time.time()

    for i in range(0, 30):
        model(input)
    t3 = time.time()

    print('%10s : %f' % (name, (t3 - t2) / 30))

if __name__ == '__main__':
    from mobilenetv1 import MobileNetV1
    mobilenetv1 = MobileNetV1().cpu()
    mobilenetv2 = MobileNetV2(width_mult=1)
    speed(mobilenetv1, 'mobilenetv1')
    speed(mobilenetv2, 'mobilenetv2')

    input = torch.randn([1, 3, 224, 224])
    output = mobilenetv1(input)
    torch.onnx.export(mobilenetv1,input,'mobilenetv1.onnx')

    output = mobilenetv2(input)
    torch.onnx.export(mobilenetv2,input,'mobilenetv2.onnx')
```
## shufflenetv1
ShuffleNetV1 是一种专门为移动设备和嵌入式系统设计的高效卷积神经网络架构。它通过引入通道混洗（Channel Shuffle）操作，有效利用了分组卷积（Group Convolution）的优势，同时避免了分组卷积导致的信息隔离问题。以下是 ShuffleNetV1 的一些关键特点：

 - 分组卷积（Group Convolution）：通过将输入通道分成多个组，分别进行卷积操作，减少计算量和模型参数。
 - 通道混洗（Channel Shuffle）：在分组卷积之后，通过通道混洗操作重新组合通道，确保信息在不同组之间充分交流。
 - 瓶颈结构（Bottleneck Structure）：使用1x1卷积进行降维和升维，中间使用3x3深度卷积进行特征提取。
### ShuffleNetV1 的基本结构
一个典型的 ShuffleNetV1 模型包括以下几个部分：
 - 输入层：接受输入图像。
 - 初始卷积层：一个标准的3x3卷积层，后面跟着批量归一化和ReLU激活。
 - 瓶颈块（Bottleneck Blocks）：多个瓶颈块，每个块包括1x1卷积、深度卷积、1x1卷积和通道混洗操作。
 - 全局平均池化层：将特征图降维为固定大小的向量。
 - 全连接层：输出分类结果。
```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import time
from collections import OrderedDict

def conv3x3(in_channels, out_channels, stride=1,padding=1, bias=True, groups=1):
    return nn.Conv2d(
        in_channels,
        out_channels,
        kernel_size=3,
        stride=stride,
        padding=padding,
        bias=bias,
        groups=groups)

def conv1x1(in_channels, out_channels, groups=1):
    return nn.Conv2d(
        in_channels,
        out_channels,
        kernel_size=1,
        groups=groups,
        stride=1)

# 通道shuffle实现
def channel_shuffle(x, groups):
    batchsize, num_channels, height, width = x.data.size()

    channels_per_group = num_channels // groups

    # reshape
    x = x.view(batchsize, groups, channels_per_group, height, width)

    # transpose
    # - contiguous() required if transpose() is used before view().
    #   See https://github.com/pytorch/pytorch/issues/764
    x = torch.transpose(x, 1, 2).contiguous()

    # flatten
    x = x.view(batchsize, -1, height, width)

    return x

# shufflenet结构单元
class ShuffleUnit(nn.Module):
    def __init__(self, in_channels, out_channels, groups=3,
                 grouped_conv=True, combine='add'):

        super(ShuffleUnit, self).__init__()

        self.in_channels = in_channels
        self.out_channels = out_channels
        self.grouped_conv = grouped_conv
        self.combine = combine
        self.groups = groups
        self.bottleneck_channels = self.out_channels // 4

        # define the type of ShuffleUnit
        if self.combine == 'add':
            # ShuffleUnit Figure 2b
            self.depthwise_stride = 1
            self._combine_func = self._add
        elif self.combine == 'concat':
            # ShuffleUnit Figure 2c
            self.depthwise_stride = 2
            self._combine_func = self._concat

            # ensure output of concat has the same channels as original output channels.
            self.out_channels -= self.in_channels
        else:
            raise ValueError("Cannot combine tensors with \"{}\"" \
                             "Only \"add\" and \"concat\" are" \
                             "supported".format(self.combine))

        # Use a 1x1 grouped or non-grouped convolution to reduce input channels
        # to bottleneck channels, as in a ResNet bottleneck module.
        # NOTE: Do not use group convolution for the first conv1x1 in Stage 2.
        self.first_1x1_groups = self.groups if grouped_conv else 1

        self.g_conv_1x1_compress = self._make_grouped_conv1x1(
            self.in_channels,
            self.bottleneck_channels,
            self.first_1x1_groups,
            batch_norm=True,
            relu=True
        )

        # 3x3 depthwise convolution followed by batch normalization
        self.depthwise_conv3x3 = conv3x3(
            self.bottleneck_channels, self.bottleneck_channels,
            stride=self.depthwise_stride, groups=self.bottleneck_channels)
        self.bn_after_depthwise = nn.BatchNorm2d(self.bottleneck_channels)

        # Use 1x1 grouped convolution to expand from bottleneck_channels to out_channels
        self.g_conv_1x1_expand = self._make_grouped_conv1x1(
            self.bottleneck_channels,
            self.out_channels,
            self.groups,
            batch_norm=True,
            relu=False
        )

    @staticmethod
    def _add(x, out):
        # residual connection
        return x + out

    @staticmethod
    def _concat(x, out):
        # concatenate along channel axis
        return torch.cat((x, out), 1)

    def _make_grouped_conv1x1(self, in_channels, out_channels, groups,
                              batch_norm=True, relu=False):

        modules = OrderedDict()

        conv = conv1x1(in_channels, out_channels, groups=groups)
        modules['conv1x1'] = conv

        if batch_norm:
            modules['batch_norm'] = nn.BatchNorm2d(out_channels)
        if relu:
            modules['relu'] = nn.ReLU()
        if len(modules) > 1:
            return nn.Sequential(modules)
        else:
            return conv

    def forward(self, x):
        # save for combining later with output
        residual = x

        if self.combine == 'concat':
            residual = F.avg_pool2d(residual, kernel_size=3,
                                    stride=2, padding=1)

        out = self.g_conv_1x1_compress(x)
        out = channel_shuffle(out, self.groups)
        out = self.depthwise_conv3x3(out)
        out = self.bn_after_depthwise(out)
        out = self.g_conv_1x1_expand(out)

        out = self._combine_func(residual, out)
        return F.relu(out)


class ShuffleNetV1(nn.Module):
    def __init__(self, groups=3, in_channels=3, num_classes=1000):
        super(ShuffleNetV1, self).__init__()

        self.groups = groups # 1*1卷积的分组参数
        self.stage_repeats = [3, 7, 3]
        self.in_channels = in_channels
        self.num_classes = num_classes

        # index 0 is invalid and should never be called.
        # only used for indexing convenience.
        if groups == 1:
            self.stage_out_channels = [-1, 24, 144, 288, 576]
        elif groups == 2:
            self.stage_out_channels = [-1, 24, 200, 400, 800]
        elif groups == 3:
            self.stage_out_channels = [-1, 24, 240, 480, 960]
        elif groups == 4:
            self.stage_out_channels = [-1, 24, 272, 544, 1088]
        elif groups == 8:
            self.stage_out_channels = [-1, 24, 384, 768, 1536]
        else:
            raise ValueError(
                """{} groups is not supported for 1x1 Grouped Convolutions""".format(groups))

        # Stage 1 always has 24 output channels
        self.conv1 = conv3x3(self.in_channels,
                             self.stage_out_channels[1],  # stage 1
                             stride=2)
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)

        # Stage 2
        self.stage2 = self._make_stage(2)
        # Stage 3
        self.stage3 = self._make_stage(3)
        # Stage 4
        self.stage4 = self._make_stage(4)

        # Fully-connected classification layer
        num_inputs = self.stage_out_channels[-1]
        self.fc = nn.Linear(num_inputs, self.num_classes)

    def _make_stage(self, stage):
        modules = OrderedDict()
        stage_name = "ShuffleUnit_Stage{}".format(stage)

        # First ShuffleUnit in the stage
        # 1. Stage 2的第一个1x1 convolution不进行分组，其他都进行分组
        grouped_conv = stage > 2

        # 2. 第一个空间分辨率下降，永远是拼接单元.
        first_module = ShuffleUnit(
            self.stage_out_channels[stage - 1],
            self.stage_out_channels[stage],
            groups=self.groups,
            grouped_conv=grouped_conv,
            combine='concat'
        )
        modules[stage_name + "_0"] = first_module

        # 后面的不用下降分辨率.
        for i in range(self.stage_repeats[stage - 2]):
            name = stage_name + "_{}".format(i + 1)
            module = ShuffleUnit(
                self.stage_out_channels[stage],
                self.stage_out_channels[stage],
                groups=self.groups,
                grouped_conv=True,
                combine='add'
            )
            modules[name] = module

        return nn.Sequential(modules)

    def forward(self, x):
        x = self.conv1(x)
        x = self.maxpool(x)

        x = self.stage2(x)
        x = self.stage3(x)
        x = self.stage4(x)

        # global average pooling layer
        x = F.avg_pool2d(x, x.data.size()[-2:])

        # flatten for input to fully-connected layer
        x = x.view(x.size(0), -1)
        x = self.fc(x)

        return F.log_softmax(x, dim=1)

def speed(model, name):
    t0 = time.time()
    input = torch.rand(1, 3, 224, 224).cpu()
    t1 = time.time()

    model(input)
    t2 = time.time()

    for i in range(0, 30):
        model(input)
    t3 = time.time()

    print('%10s : %f' % (name, (t3 - t2) / 30))

if __name__ == "__main__":
    model = ShuffleNetV1()
    input = torch.rand(1,3,224,224).cpu()
    output = model(input)
    speed(model, 'shufflenet v1')
    #print(model)

```

## shufflenetv2
ShuffleNetV2 是 ShuffleNetV1 的改进版本，旨在进一步提高模型的效率和准确性。ShuffleNetV2 通过一系列实验和理论分析，提出了一套更有效的设计原则，并在此基础上构建了新的网络架构。以下是 ShuffleNetV2 的一些关键特点：

 - 平衡计算负载：确保每个阶段的计算负载均匀分布，避免某些层的计算量过大。
 - 避免信息丢失：通过设计合理的跳跃连接，确保信息在不同层之间有效传递。
 - 减少内存访问成本：通过优化数据布局和操作顺序，减少内存访问次数，提高运行效率。
 - 通道混洗（Channel Shuffle）：在分组卷积之后，通过通道混洗操作重新组合通道，确保信息在不同组之间充分交流。
### ShuffleNetV2 的基本结构
一个典型的 ShuffleNetV2 模型包括以下几个部分：
 - 输入层：接受输入图像。
 - 初始卷积层：一个标准的3x3卷积层，后面跟着批量归一化和ReLU激活。
 - 基本块（Basic Blocks）：多个基本块，每个块包括1x1卷积、深度卷积、1x1卷积和通道混洗操作。
 - 全局平均池化层：将特征图降维为固定大小的向量。
 - 全连接层：输出分类结果。
```python
import torch
import torch.nn as nn
import time

def channel_shuffle(x, groups):
    batchsize, num_channels, height, width = x.data.size()
    channels_per_group = num_channels // groups

    # reshape
    x = x.view(batchsize, groups,
               channels_per_group, height, width)

    x = torch.transpose(x, 1, 2).contiguous()

    # flatten
    x = x.view(batchsize, -1, height, width)

    return x


class InvertedResidual(nn.Module):
    def __init__(self, inp, oup, stride):
        super(InvertedResidual, self).__init__()

        if not (1 <= stride <= 3):
            raise ValueError('illegal stride value')
        self.stride = stride

        branch_features = oup // 2
        assert (self.stride != 1) or (inp == branch_features << 1)

        if self.stride > 1:
            self.branch1 = nn.Sequential(
                self.depthwise_conv(inp, inp, kernel_size=3, stride=self.stride, padding=1),
                nn.BatchNorm2d(inp),
                nn.Conv2d(inp, branch_features, kernel_size=1, stride=1, padding=0, bias=False),
                nn.BatchNorm2d(branch_features),
                nn.ReLU(inplace=True),
            )

        self.branch2 = nn.Sequential(
            nn.Conv2d(inp if (self.stride > 1) else branch_features,
                      branch_features, kernel_size=1, stride=1, padding=0, bias=False),
            nn.BatchNorm2d(branch_features),
            nn.ReLU(inplace=True),
            self.depthwise_conv(branch_features, branch_features, kernel_size=3, stride=self.stride, padding=1),
            nn.BatchNorm2d(branch_features),
            nn.Conv2d(branch_features, branch_features, kernel_size=1, stride=1, padding=0, bias=False),
            nn.BatchNorm2d(branch_features),
            nn.ReLU(inplace=True),
        )

    @staticmethod
    def depthwise_conv(i, o, kernel_size, stride=1, padding=0, bias=False):
        return nn.Conv2d(i, o, kernel_size, stride, padding, bias=bias, groups=i)

    def forward(self, x):
        if self.stride == 1:
            x1, x2 = x.chunk(2, dim=1)
            out = torch.cat((x1, self.branch2(x2)), dim=1)
        else:
            out = torch.cat((self.branch1(x), self.branch2(x)), dim=1)

        out = channel_shuffle(out, 2)

        return out


class ShuffleNetV2(nn.Module):
    def __init__(self, stages_repeats, stages_out_channels, num_classes=1000):
        super(ShuffleNetV2, self).__init__()

        if len(stages_repeats) != 3:
            raise ValueError('expected stages_repeats as list of 3 positive ints')
        if len(stages_out_channels) != 5:
            raise ValueError('expected stages_out_channels as list of 5 positive ints')
        self._stage_out_channels = stages_out_channels

        input_channels = 3
        output_channels = self._stage_out_channels[0]
        self.conv1 = nn.Sequential(
            nn.Conv2d(input_channels, output_channels, 3, 2, 1, bias=False),
            nn.BatchNorm2d(output_channels),
            nn.ReLU(inplace=True),
        )
        input_channels = output_channels

        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)

        stage_names = ['stage{}'.format(i) for i in [2, 3, 4]]
        for name, repeats, output_channels in zip(
                stage_names, stages_repeats, self._stage_out_channels[1:]):
            seq = [InvertedResidual(input_channels, output_channels, 2)]
            for i in range(repeats - 1):
                seq.append(InvertedResidual(output_channels, output_channels, 1))
            setattr(self, name, nn.Sequential(*seq))
            input_channels = output_channels

        output_channels = self._stage_out_channels[-1]
        self.conv5 = nn.Sequential(
            nn.Conv2d(input_channels, output_channels, 1, 1, 0, bias=False),
            nn.BatchNorm2d(output_channels),
            nn.ReLU(inplace=True),
        )

        self.fc = nn.Linear(output_channels, num_classes)

    def forward(self, x):
        x = self.conv1(x)
        x = self.maxpool(x)
        x = self.stage2(x)
        x = self.stage3(x)
        x = self.stage4(x)
        x = self.conv5(x)
        x = x.mean([2, 3])  # globalpool
        x = self.fc(x)
        return x

def speed(model, name):
    t0 = time.time()
    input = torch.rand(1, 3, 224, 224).cpu()
    t1 = time.time()

    model(input)
    t2 = time.time()

    for i in range(0, 30):
        model(input)
    t3 = time.time()

    print('%10s : %f' % (name, (t3 - t2) / 30))

if __name__ == "__main__":
    from shufflenetv1 import ShuffleNetV1

    shufflenetv1 = ShuffleNetV1()
    shufflenetv2 = ShuffleNetV2([3, 7, 3], [24, 116, 232, 464, 1024])

    speed(shufflenetv1, 'shufflenet v1')
    speed(shufflenetv2, 'shufflenet v2')

    input = torch.randn([1, 3, 224, 224])
    output = shufflenetv1(input)
    torch.onnx.export(shufflenetv1,input,'shufflenetv1.onnx')

    output = shufflenetv2(input)
    torch.onnx.export(shufflenetv2,input,'shufflenetv2.onnx')


```

