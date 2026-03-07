---
title: Python：pip 安装第三方库速度很慢的解决办法，以及离线安装方法和conda换源，以及指定路径打开jupyter notebook
date: 2026-01-02 08:16:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿## 微信公众号：[数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&amp;mid=100000040&amp;idx=1&amp;sn=c4d782027e72eec3ac189d856c5ecd6a&amp;scene=19&token=1933614188&lang=zh_CN#wechat_redirect)
@[TOC](Python：pip 安装第三方库速度很慢的解决办法，以及离线安装方法和conda换源)
# pip
**举个例子，比如想要安装 mlxtend 库**

键盘按着 win + R键，输入cmd并点击确定，打开命令行窗口；在 cmd 敲入命令

```csharp
pip install mlxtend
```
但是发现下载安装文件非常慢，有可能出现警告，还有可能最后显示当下载到百分之几十的时候窗口中就会出现一堆红字，最后安装失败。

**原因**：在命令行窗口内输入pip help install，查看帮助文档，向下滑动可以看到`-i, --index-url <url> ...`选项，文档中可以看到python默认安装源的地址实质访问的下载网站是[https://pypi.Python.org/simple/](https://pypi.Python.org/simple/)  。因为这是一个国外网站，所以在国内下载速度比较慢。
## 方法一
使用国内源

在 cmd 更改为敲入命令（**pip install -i https://pypi.tuna.tsinghua.edu.cn/simple 库**）

清华大学源

```csharp
pip install mlxtend -i https://pypi.tuna.tsinghua.edu.cn/simple
```

或者命令行中输入

```csharp
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple mlxtend
```

以后使用 pip 安装第三方包时，都可以把**-i https://pypi.douban.com/simple** 作为必填的内容，这样速度会变得很快。

## 方法二
第二种就是一劳永逸的方法，选择配置国内镜像源。

**Windows** 首先找到C:\Users\Solitude\AppData\Roaming，这个路径的文件夹，如果你没有找到，那就是你的文件夹被隐藏了，解决办法如下：
打开C盘，点击左上角的“查看”-“选项”- 显示/隐藏 中勾选“隐藏的项目”，然后确定即可。这样你就能看到AppData文件夹了。（如下图）
![在这里插入图片描述](/img/49adb77c0e065456a615016a7f399a9d.png)
找到路径后，首选在该路径下新建文件夹，命名为“pip”，然后在pip文件夹中新建一个txt格式的文本文档，打开文本文档，将下面这些代码复制到文本文档中，关闭保存。然后将txt格式的文本文档重新命名为“pip.ini”，这样就创建了一个配置文件。

```python
[global]
timeout = 60000
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
use-mirrors = true
mirrors = https://pypi.tuna.tsinghua.edu.cn
```
文档中的链接地址还可以更换其他的如下：
阿里云 [http://mirrors.aliyun.com/pypi/simple/](http://mirrors.aliyun.com/pypi/simple/)
中国科技大学 [https://pypi.mirrors.ustc.edu.cn/simple/](https://pypi.mirrors.ustc.edu.cn/simple/)
豆瓣(douban) [http://pypi.douban.com/simple/](http://pypi.douban.com/simple/)
清华大学 [https://pypi.tuna.tsinghua.edu.cn/simple/](https://pypi.tuna.tsinghua.edu.cn/simple/)
中国科学技术大学 [http://pypi.mirrors.ustc.edu.cn/simple/](http://pypi.mirrors.ustc.edu.cn/simple/)
这样再使用pip进行包安装时候就默认选择国内源进行安装了，速度超快！！！

**Linux/macOS**: 配置文件路径为 ~/.pip/pip.conf 或 ~/.config/pip/pip.conf。 如果文件不存在，可以手动创建。
# 本地离线安装方法
查找第三方包可以从以下网址查找：
1、[PyPI · The Python Package Index](https://pypi.org/)
​![在这里插入图片描述](/img/8e7b93ce779edacf760225ab93d758ac.png)

2、[Archived: Python Extension Packages for Windows - Christoph Gohlke (uci.edu)
​](https://www.lfd.uci.edu/~gohlke/pythonlibs/)
![在这里插入图片描述](/img/201e0005dcdaee3a54bb1fc4c33aba9a.png)

 1. 如果你下载的是whl文件，下载完后放到你先要安装的位置文件夹，在所在位置打开cmd，使用pip install 文件名（包括whl后缀）
 2. 如果你下载的是压缩包，直接将解压后的文件夹放入到你想安装的位置，一般是放到之前安装的库一起，然后打开文件夹，进入cmd，输入`python setup.py install`

# Conda 换源步骤
## 查看当前配置

```bash
conda config --show channels
```
## 添加国内镜像源
**清华大学镜像源**

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
```
**阿里云镜像源**

```bash
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/main/
conda config --add channels https://mirrors.aliyun.com/anaconda/pkgs/free/
```
## 设置默认镜像源

```bash
conda config --set show_channel_urls yes
```

# 恢复 Conda 默认源
如果你想要恢复 Conda 到默认的官方源，可以按照以下步骤操作：
## 删除自定义的镜像源

```bash
conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --remove channels https://mirrors.aliyun.com/anaconda/pkgs/main/
conda config --remove channels https://mirrors.aliyun.com/anaconda/pkgs/free/
```
## 重置为默认源

```bash
conda config --remove-key channels
conda config --add channels defaults
```
# 指定路径打开jupyter notebook
在启动 Jupyter Notebook 之前，先切换到 D 盘的某个目录：

```bash
cd /d D:\your_directory
```
然后再输入
```bash
jupyter notebook
```
即可
