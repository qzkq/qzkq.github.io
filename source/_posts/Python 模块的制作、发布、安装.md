---
title: Python 模块的制作、发布、安装
date: 2026-01-02 08:03:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Python 模块的制作、发布、安装)

# 作用：

 - 可以使我们有逻辑的去组织我们的python代码
 - 以库的形式去封装功能，非常方便的去让调用者去使用
 - 可以定义函数、类、变量，也能包含可执行的代码

**注**：不同的模块可以定义相同的变量名，但是每个模块中的变量名作用域只是在本模块中
# 1.模块的制作
•（1）**Python文件都可以作为一个模块，模块的名字就是文件的名字。** 比如创建一个test.py文件，文件中创建一个add函数。test.py就是一个模块。
![在这里插入图片描述](/img/0e37a71831fd42969131e2cda6652264.png)

•（2）调用test.py模块

![在这里插入图片描述](/img/3282176968384c7187e8cf638a348e49.png)

•（3）模块测试 一般写完模块之后都会进行测试，下面来看下这个例子 写好模块之后，在模块中写了一段测试的代码。

![在这里插入图片描述](/img/42f8aaca6f264cbdb5dbae29d15d5891.png)

•在main.py 导入test模块，执行时模块中的测试代码也执行了。

![在这里插入图片描述](/img/19a1076351aa4c99971d52b872633063.png)

•（4）为了避免这种情况使用到一个__name__的变量。

![在这里插入图片描述](/img/c5050b7ab8734217880634665b8c78ce.png)

•在main.py中导入执行

![在这里插入图片描述](/img/6706f4967fc3417a94eb69a293f14f9c.png)

•知道__name__变量的原理之后，就可以很好的处理测试代码了。 将测试的代码放到 `if __name__ = '__main__'：`

![在这里插入图片描述](/img/fa3448c332ce4052b9c36385306f5ddb.png)

•（5）`__all__` 的作用，如果一个文件中有__all__变量，那么也就意味着这个变量中的元素，会被from xxx import * 时导入

![在这里插入图片描述](/img/4fa2ff599d9d4ce78f75d84618292aff.png)

•有all变量import方式导入，可以无异常，可以正常使用。

![在这里插入图片描述](/img/650dba4c015045c19cfa85fb1e0b935f.png)

•from test import * 方式导入

![在这里插入图片描述](/img/4f31be49075c4cc4971a4d8803e69223.png)

从例子可以看出： 使用from xxx import 导入方式，不在__all__变量中是无法导入的，其他导入方式不影响。
# 2.模块的发布
•平时使用第三方模块是其他开发者发布出来，需要安装后调用。发布一个模块的步骤：
•（1）将写好的包放到一个test/目录下

![在这里插入图片描述](/img/2837f791ca6b4550858b25d7bb13c7f5.png)

•（2）在test/目录下创建一个文件setup.py文件
•文件里写入下面代码

```bash
from distutils.core import setup 
# name 模块名称 
# version 版本号 
# description 描述 
# author 作者 
# py_modules 要发布的内容 
setup(name="my_module", version="1.0", description="my module", 
author="lilei", py_modules=['test1.A', 'test1.B', 'test2.C', 'test2.D'])
```

 •（3）创建模块
通过终端进入当前文件目录

![在这里插入图片描述](/img/c4c836563670433d99d051f713d51fdd.png)

•python setup.py build

![在这里插入图片描述](/img/dd61233fc8ea474ab3934a61266537b9.png)

•（4）生成压缩包
•python setup.py sdist

![在这里插入图片描述](/img/9f3ba042666d417db0c12274f0e15355.png)

•（5）看下test目录下的结构

![在这里插入图片描述](/img/a92173675d4b47328e3310f0d6fbedca.png)

# 3.模块的安装
•（1）将上一节生成的压缩包复制到桌面解压
•tar -zxvf qzk-1.0.tar.gz
•解压后在桌面会生成一个文件夹qzk-1.0
•（2）进入qzk-1.0文件夹
•cd qzk-1.0-1.0
•（3）执行命令安装 python setup.py install 
•（4）查看是否安装成功
在python的安装目录下的site-packages目录下
**或者**
为了方便我们在当前目录安装压缩包
•pip install  qzk-1.0.tar.gz

![在这里插入图片描述](/img/22e8bf73ee6d464db47e98b68cab812c.png)

•模块引入，可以导入说明安装成功

# 示例

制作、发布和安装 Python 模块的过程可以分为几个步骤，这里以一个假设的模块 clearsky 为例来说明整个流程：
1. 创建模块
首先，你需要创建一个包含你的代码的文件夹，比如叫做 clearsky。在这个文件夹里，你应该有以下文件结构：

```bash
clearsky/
│
├── clearsky/
│   ├── __init__.py
│   └── your_code.py
│
├── setup.py
└── README.md

```

 - clearsky/: 包含你的代码文件。
 - __init__.py: 这个文件可以为空，它的存在告诉 Python 这个目录应该被当作一个包处理
 - your_code.py: 包含你的功能实现。
 - setup.py: 用于配置打包和发布信息。
 - README.md: 介绍你的模块。
2. 编写 setup.py
setup.py 是一个重要的文件，它包含了关于你的模块的信息以及如何构建、打包和发布你的模块。一个简单的 setup.py 可能如下所示：

```bash
from setuptools import setup, find_packages

setup(
    name='clearsky',
    version='0.1',
    packages=find_packages(),
    install_requires=[
        # 依赖项列表
    ],
    author='Your Name',
    author_email='your.email@example.com',
    description='A short description of the package',
    long_description=open('README.md').read(),
    long_description_content_type='text/markdown',
    url='https://github.com/yourusername/clearsky',
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)

```
3. 发布到 PyPI
在发布之前，确保你已经注册了 PyPI 账号，并且安装了 twine 和 wheel 工具：

```bash
pip install twine wheel

```
然后，你可以通过以下命令来打包并发布你的模块：

```bash
python setup.py sdist bdist_wheel
twine upload dist/*

```
4. 安装模块
一旦模块发布成功，其他人就可以通过 pip 来安装你的模块：

```bash
pip install clearsky

```
这样，他们就可以在自己的项目中导入并使用你的模块了。
