---
title: 如何使用Cython对python脚本加密成pyd_so
date: 2026-01-04 08:08:00
tags:
  - "Python"
  - "代码加密"
  - "Cython"
  - "pyd"
  - "so"
categories:
  - "开发工具"
---

﻿@[TOC](Microsoft Visual C++ 14.0 or greater is required. Get it with “Microsoft C++ Build Tools“的解决办法)
## 第一步: 安装Cython
在开始使用Cython编译Python代码之前，您需要先安装Cython。您可以使用pip来安装Cython，可以在命令行界面中输入以下命令（一般python自带）:

```shell
pip install cython
```

## 第二步：编写Python代码并使用Cython编译
示例hello.py

```python
def hello():
    print("Hello World!")
```

为了编译Python代码为Cython模块，我们需要编写一个`setup.py`文件。在Python的安装目录下，创建一个新文件夹并命名为cython_example。在该文件夹下创建一个名为setup.py的文件，文件内容如下：

```python
from distutils.core import setup
from Cython.Build import cythonize

setup(name='hello',
      ext_modules=cythonize("hello.py"))
```

在命令行界面中的Python安装路径下，运行以下命令来编译Python代码并生成Cython模块：

```powershell
python setup.py build_ext --inplace
```

这将会在Python安装路径下生成一个新的文件`hello.cp39-win_amd64.pyd`（使用Python 3.9发行版的Windows操作系统），该文件包含编译的Python代码并可以被导入到其他Python代码中。**windows下为pyd文件，linux下为so文件**

## 第三步: 使用Cython模块导入加密的Python代码
现在我们已经编译了加密的Python代码，接下来将代码导入到其他Python代码中。假设我们有一个名为app.py的Python文件，我们希望在其中调用hello.py中的hello()函数。

```python
from hello import hello

hello()
```

此时我们可以启动Python解释器，运行app.py，输出结果应该是`“Hello World!”`。

## 在Windows系统上将py加密成pyd时或者使用pip安装一些软件时，会出现下面这样的问题

```powershell
error: Microsoft Visual C++ 14.0 or greater is required. Get it with “Microsoft C++ Build Tools”: https://visualstudio.microsoft.com…
```
如果按照错误提示的信息来做，那么会引导安装Visual Studio。但是一方面安装Visual Studio需要时间很久，另外一方面会占用大量的磁盘空间，让空间原本就不富裕的固态硬盘雪上加霜。
### 解决方案：
直接安装Microsoft C++ Build Tools，而不需要安装Visual Studio。
[Visual Studio Subscriptions](https://my.visualstudio.com/Downloads?q=Build%20Tools%20for%20Visual%20Studio%202017%20%28version%2015.0%29)

在下载页面搜索Build Tools for Visual Studio 2015；进行安装。

![在这里插入图片描述](/img/a6dd52027e7a10deb2c0ce601a4da009.png)

