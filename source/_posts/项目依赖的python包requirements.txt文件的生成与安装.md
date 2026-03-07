---
title: 项目依赖的python包requirements.txt文件的生成与安装
date: 2026-01-05 08:19:00
tags:
  - "Python"
  - "pip"
  - "requirements.txt"
  - "依赖管理"
categories:
  - "开发工具"
---

﻿[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
@[TOC](项目依赖的python包requirements.txt文件的生成与安装)
## 项目依赖的python包requirements.txt文件的生成与安装
在使用python进行项目开发的时候常常会调用许多包，而这些包又是在不停的更新中的。因此，当前项目所需要的包的功能，在以后包的迭代中可能会被取代或者更新，从而导致在以后的某个时间重启项目的时候无法运行。所以记录下当项目所需要的包的类型以及版本是非常重要的，方便以后重启项目的时候可以直接安装。

**requirements.txt是定义项目依赖的python包，可通过工具生成。工具可以生成两种依赖包定义，一是项目依赖的python包，二是所在python环境安装的python包。**

### 一、生成requirements.txt文件
#### 1. 生成项目依赖包步骤（推荐）
安装pipreqs工具，命令：`pip3 install pipreqs` 

到项目根目录下，命令： `pipreqs ./`  （若出现编码错误，则可使用：pipreqs ./ --encoding=utf8  ; 若已存在requirements.txt,则可使用--force 强制执行）

这时会生成requirements.txt文件
![在这里插入图片描述](/img/d7bf2de501b79e0fd6a4b26139ecb888.png)

#### 2. 生成整个当前python环境安装的python包（全局环境）
这种方式是会在当前路径下生成一个requirements.txt文件，该文件中则会记录当前python环境下所以拥有的所有包，以及包的版本。可以看作把pip list这个命令展现的所有东西记录下了。这种方式速度很快，但是requirements.txt文件包含的包是当前环境所有的包，如果你当前项目没有用到的包也会被包括下来。

到项目根目录下，直接运行：`pip3 freeze > requirements.txt`
![在这里插入图片描述](/img/26a0c97c0465bdd09d3f32a5b149faa2.png)
### 二、安装requirements.txt文件
推荐采用conda新建一个虚拟环境之后再使用以下命令

```bash
pip3 install -r requirements.txt
```

添加下载镜像，来加速下载
　
```
pip3 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**注：**
如果自己有项目所在的源环境（比如你是自己从一个电脑复制到另一个电脑）可以把源环境粘贴复制即可。
如出现以下错误：

```csharp
Fatal error in launcher: Unable to create process using '"C:\Users|Administrator\Anaconda\envs\python\python.exe""D:\Anaconda\envs\python\Scripts\pip.exe"'
```

可以重新安装pip，如果没有网络，想要离线安装whl文件，可以在pip install前面加python -m

```csharp
python -m pip install xxx.whl
```
(直接复制虚拟环境的话如果还有问题，用notepad++打开Scripts文件夹下的pip.exe, 全局搜索C:\，找到python.exe路径，修改成自己的虚拟环境名)
