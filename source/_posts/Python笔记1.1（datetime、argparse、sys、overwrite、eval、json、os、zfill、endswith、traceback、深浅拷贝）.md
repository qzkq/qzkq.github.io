---
title: Python笔记1.1（datetime、argparse、sys、overwrite、eval、json、os、zfill、endswith、traceback、深浅拷贝）
date: 2026-01-02 08:11:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿[Python笔记2（函数参数、面向对象、装饰器、高级函数、捕获异常、dir）](https://blog.csdn.net/qq_45832050/article/details/134106632)
@[TOC](Python笔记：datetime、argparse、sys、overwrite、eval、json、os、zfill、endswith、traceback、深浅拷贝)
## 1、datetime之字符串日期互相转换
### 主要类
**datetime.datetime**
表示具体的日期和时间。可以用来进行日期和时间的计算。
**datetime.date**
仅表示日期（年、月、日）。
**datetime.time**
仅表示时间（小时、分钟、秒、微秒）。
**datetime.timedelta**
表示两个日期或时间之间的差值。
### 常用方法
#### datetime.datetime
**datetime.now()**
获取当前本地日期和时间。

```python
from datetime import datetime
now = datetime.now()
print(now)

```
**datetime.today()**
同 datetime.now()，获取当前本地日期和时间。

```python
from datetime import datetime
today = datetime.today()
print(today)
    
```
**datetime.utcnow()**
获取当前 UTC 时间。

```python
from datetime import datetime
utc_now = datetime.utcnow()
print(utc_now)
    
```
**datetime.strptime()**
将字符串转换为日期时间对象。

```python
from datetime import datetime
date_str = "2023-04-01"
date_obj = datetime.strptime(date_str, "%Y-%m-%d")
print(date_obj)
    
```
**datetime.strftime()**
将日期时间对象转换为字符串。

```python
from datetime import datetime
now = datetime.now()
date_str = now.strftime("%Y-%m-%d %H:%M:%S")
print(date_str)
    
```
#### datetime.date
**date.today()**
获取当前日期。

```python
from datetime import date
today = date.today()
print(today)
    
```
**date.fromisoformat()**
从 ISO 格式的字符串创建日期对象。

```python
from datetime import date
date_str = "2023-04-01"
date_obj = date.fromisoformat(date_str)
print(date_obj)
    
```
#### datetime.time
**time()**
创建一个时间对象。

```python
from datetime import time
t = time(hour=12, minute=30, second=45)
print(t)
    
```
#### datetime.timedelta
**timedelta()**
创建一个时间间隔对象。

```python
from datetime import timedelta
delta = timedelta(days=5, hours=3, minutes=30)
print(delta)
    
```
**加减操作**
可以对日期和时间对象进行加减操作。

```python
from datetime import datetime, timedelta
now = datetime.now()
future = now + timedelta(days=5)
past = now - timedelta(hours=3)
print(future)
print(past)
    
```
### 格式化字符串

```python
%Y 年份（四位数）
%y 年份（两位数）
%m 月份（01-12）
%d 日（01-31）
%H 小时（24小时制，00-23）
%I 小时（12小时制，01-12）
%M 分钟（00-59）
%S 秒（00-59）
```
## 2、argparse
argparse 是 Python 的标准库之一，用于命令行参数解析。它可以轻松地编写用户友好的命令行接口。
### 基本用法

 1. 创建 ArgumentParser 对象
 2. 添加参数
 3. 解析命令行参数

```python
import argparse

# 创建 ArgumentParser 对象
parser = argparse.ArgumentParser(description="一个简单的命令行程序")

# 添加参数
parser.add_argument("name", help="你的名字")
parser.add_argument("-a", "--age", type=int, help="你的年龄")

# 解析命令行参数
args = parser.parse_args()

# 输出结果
print(f"你好，{args.name}！")
if args.age:
    print(f"你的年龄是 {args.age} 岁。")

```
### 参数类型
argparse 支持多种参数类型，如字符串、整数、浮点数等。

### 位置参数和可选参数

 - 位置参数：必须按顺序传递。
 - 可选参数：可以省略，默认值通常为 None 或者自定义默认值。

```python
import argparse

parser = argparse.ArgumentParser(description="一个简单的命令行程序")
parser.add_argument("name", help="你的名字")
parser.add_argument("-a", "--age", type=int, default=18, help="你的年龄")
parser.add_argument("-g", "--gender", choices=["male", "female"], help="性别")

args = parser.parse_args()

print(f"你好，{args.name}！")
print(f"你的年龄是 {args.age} 岁。")
if args.gender:
    print(f"你的性别是 {args.gender}。")

```

### 互斥组
互斥组允许设置一组互斥的选项，即只能选择其中一个。

```python
import argparse

parser = argparse.ArgumentParser(description="一个简单的命令行程序")
group = parser.add_mutually_exclusive_group()
group.add_argument("-v", "--verbose", action="store_true", help="详细模式")
group.add_argument("-q", "--quiet", action="store_true", help="安静模式")

args = parser.parse_args()

if args.verbose:
    print("详细模式开启！")
elif args.quiet:
    print("安静模式开启！")
else:
    print("正常模式。")

```

### 帮助信息
argparse 自动生成帮助信息，可以通过 -h 或 --help 查看。

```python
import argparse

parser = argparse.ArgumentParser(description="一个简单的命令行程序")
parser.add_argument("name", help="你的名字")
parser.add_argument("-a", "--age", type=int, default=18, help="你的年龄")
parser.add_argument("-g", "--gender", choices=["male", "female"], help="性别")

args = parser.parse_args()

print(f"你好，{args.name}！")
print(f"你的年龄是 {args.age} 岁。")
if args.gender:
    print(f"你的性别是 {args.gender}。")

```
运行命令

```bash
python script.py -h

```
输出

```python
usage: script.py [-h] [-a AGE] [-g {male,female}] name

一个简单的命令行程序

positional arguments:
  name              你的名字

optional arguments:
  -h, --help        show this help message and exit
  -a AGE, --age AGE
                    你的年龄
  -g {male,female}, --gender {male,female}
                    性别

```

## 3、sys
sys 是 Python 的一个内置模块，它提供了访问和改变 Python 运行时环境的方法。sys 模块主要包含了一系列与 Python 解释器相关的变量和函数，使得开发者能够与解释器进行交互。
**常见成员**

 - sys.argv: 包含了命令行参数列表，第一个参数通常是脚本名称（即程序本身）。可以通过这个列表来获取传递给脚本的参数。
 - sys.exit(n=0):
   当调用此函数时，解释器将终止当前程序的执行，并且可以选择返回一个状态码给操作系统。默认的状态码为0，表示成功。
 - sys.path: 是一个字符串列表，包含了模块搜索路径。解释器会在这些路径中查找导入的模块。
 - sys.stdin, sys.stdout, sys.stderr:
   分别对应标准输入、标准输出和标准错误流。可以用来读取输入或者重定向输出。
 - sys.version: 返回 Python 解释器的版本信息作为字符串。
 - sys.platform: 返回解释器所在的平台，例如 'win32', 'darwin', 'linux' 等。
 - sys.modules: 是一个字典，包含了所有已经导入的模块。
 - sys.exc_info 用于获取当前正在处理的异常信息。它返回一个三元组，包含异常类型、异常实例和 traceback 对象。

```python
import sys

# 获取命令行参数
args = sys.argv[1:]

# 输出参数
print("命令行参数:", args)

# 示例：运行脚本
# python script.py arg1 arg2 arg3

```

## 4、overwrite
dataframe写入的一种模式，dataframe写入的模式一共有4种

```bash
def mode(saveMode: String): DataFrameWriter = {
    this.mode = saveMode.toLowerCase match {
      case "overwrite" => SaveMode.Overwrite              // 覆盖已经存在的文件
      case "append" => SaveMode.Append                    // 向存在的文件追加
      case "ignore" => SaveMode.Ignore                    // 如果文件已存在，则忽略保存操作
      case "error" | "default" => SaveMode.ErrorIfExists  // 如果文件存在，则报错
      case _ => throw new IllegalArgumentException(s"Unknown save mode: $saveMode. " +
        "Accepted modes are 'overwrite', 'append', 'ignore', 'error'.")
    }
    this
  }
```
## 5、eval
用来执行一个字符串表达式，并返回表达式的值

```python
eval(expression[, globals[, locals]])
```
参数
 - expression -- 表达式。 
 - globals -- 变量作用域，全局命名空间，如果被提供，则必须是一个字典对象。 
 - locals   -- 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。

**还可以进行字符串与list、tuple、dict的转化**

## 6、json.dumps()和json.loads()

 1. json.dumps()：将 Python 对象 转换为 JSON 格式的字符串。
 2. json.loads()：将 JSON 格式的字符串 解析为 Python 对象（通常是字典或列表）。
#### json.dumps()关键参数：

- `indent`：缩进（美化输出，例如 `indent=4`）
- `sort_keys`：按键排序（`sort_keys=True`）
- `ensure_ascii`：处理非 ASCII 字符（设为 `False` 可保留中文等）
#### 类型映射关系

| JSON 类型      | Python 类型    |
| :------------- | :------------- |
| `object`       | `dict`         |
| `array`        | `list`         |
| `string`       | `str`          |
| `number`       | `int`/`float`  |
| `true`/`false` | `True`/`False` |
| `null`         | `None`         |


## 7、os.system(cmd)
使用os.system(cmd)即可在python中使用linux命令

```python
import os

commands = [
    "pwd",
    "ls",
    "mkdir new_directory",
    "echo 'Hello, World!' > test_file.txt",
    "cat test_file.txt",
    "rm test_file.txt"
]

for cmd in commands:
    result = os.system(cmd)
    print(f"执行命令 '{cmd}' 的结果: {result}")

```

## 8、if \_\_name__ == '_\_main__':的作用
一个python文件通常有两种使用方法，第一是作为脚本直接执行，第二是 import 到其他的 python 脚本中被调用（模块重用）执行。因此 if \_\_name\_\_ == 'main': 的作用就是控制这两种情况执行代码的过程，在 if \_\_name__ == 'main': 下的代码只有在第一种情况下（即文件作为脚本直接执行）才会被执行，而 import 到其他脚本中是不会被执行的。

## 9、zfill
`str.zfill(width)` 方法会在字符串左侧填充零，直到字符串的总长度达到 width。
## 10、如果不够两位，前位补0
```python
# 如果不够两位，前位补0
conf = "qzk"
date = "20220606"
h = 6
h1 = 16
s1 = f"{conf}/{date}/{h:0>2d}"
s2 = f"{conf}/{date}/{h1:0>2d}"
print(s1)
print(s2)
# 输出
qzk/20220606/06
qzk/20220606/16
```

## 11、Python 直接赋值、浅拷贝和深度拷贝解析

 - 直接赋值：其实就是对象的引用（别名）。
 - 浅拷贝(copy)：拷贝父对象，不会拷贝对象的内部的子对象。
 - 深拷贝(deepcopy)： copy 模块的 deepcopy 方法，完全拷贝了父对象及其子对象。

字典浅/深拷贝实例
```python
>>> a = {1: [1,2,3]}
>>> b = a.copy()
>>> a, b
({1: [1, 2, 3]}, {1: [1, 2, 3]})
>>> a[1].append(4)
>>> a, b
({1: [1, 2, 3, 4]}, {1: [1, 2, 3, 4]})

>>> import copy
>>> c = copy.deepcopy(a)
>>> a, c
({1: [1, 2, 3, 4]}, {1: [1, 2, 3, 4]})
>>> a[1].append(5)
>>> a, c
({1: [1, 2, 3, 4, 5]}, {1: [1, 2, 3, 4]})
```
解析

```sql
1、b = a: 赋值引用，a 和 b 都指向同一个对象。
2、b = a.copy(): 浅拷贝, a 和 b 是一个独立的对象，但他们的子对象还是指向统一对象（是引用）。
3、b = copy.deepcopy(a): 深度拷贝, a 和 b 完全拷贝了父对象及其子对象，两者是完全独立的。
```

## 12、endswith()

作用：判断字符串是否以指定字符或子字符串结尾，常用于判断文件类型
相关函数：判断字符串开头 startswith()

**函数说明**语法：
```python
string.endswith(str, beg=[0,end=len(string)])
string[beg:end].endswith(str)
```
**参数说明**：

 - string： 被检测的字符串
 - str： 指定的字符或者子字符串（可以使用元组，会逐一匹配）
 - beg： 设置字符串检测的起始位置（可选，从左数起）
 - end： 设置字符串检测的结束位置（可选，从左数起）

如果存在参数 beg 和 end，则在指定范围内检查，否则在整个字符串中检查
返回值：
如果检测到字符串，则返回True，否则返回False。

解析：如果字符串string是以str结束，则返回True，否则返回False

注：会认为空字符为真

## 13、traceback 
用于处理 Python 解释器内部的堆栈跟踪信息。当程序中抛出一个未被处理的异常时，Python 解释器会打印一个 traceback，这是一个错误报告，包含了导致程序崩溃的异常信息以及异常发生时的调用堆栈。
**基本概念**

 - Traceback：当一个异常没有被捕获时，Python 会打印一个 traceback，显示异常发生的位置以及异常的类型和值。
 - 堆栈调用：traceback 显示了从异常发生点到程序开始执行的一系列函数调用，通常是从最近的调用到最远的调用，即从顶部到底部。
 - 异常：是程序执行过程中遇到的问题，如除零错误、类型错误等。

**常用方法**

 - traceback.print_tb(tb, limit=None, file=None)：打印 traceback
   对象到指定的文件对象，默认为 sys.stderr。
 - traceback.print_exception(etype, value, tb, limit=None,
   file=None)：打印异常类型、值及 traceback 到指定的文件对象。
 - traceback.format_tb(tb, limit=None)：返回一个字符串列表，每个字符串代表一个栈帧。
 - traceback.format_exception(etype, value, tb,
   limit=None)：返回一个字符串列表，这些字符串可以打印来显示完整的异常信息。
 - traceback.extract_tb(tb, limit=None)：返回一个列表，每个元素是一个四元组 (filename,  line number, function name, text)，表示一个栈帧。

```python
 try:
      func1()
  except Exception as e:
      tb = traceback.extract_tb(sys.exc_info()[2])
      
traceback.extract_tb 返回一个列表，其中每个元素是一个四元组，包含以下信息：
	filename: 异常发生的文件名。
	lineno: 异常发生的行号。
	name: 当前栈帧的函数名。
	line: 当前行的源代码（如果可用）。
```

