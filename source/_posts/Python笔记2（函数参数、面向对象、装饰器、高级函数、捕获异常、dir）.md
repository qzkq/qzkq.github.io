---
title: Python笔记2（函数参数、面向对象、装饰器、高级函数、捕获异常、dir）
date: 2026-01-02 08:13:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿[Python笔记1（赋值、浅拷贝和深拷贝、字符串日期转换、argparse、sys、overwrite、eval、json.dumps/json.loads、os.system(cmd)、zfill、endswith、open）](https://blog.csdn.net/qq_45832050/article/details/126790003)
@[TOC](Python笔记2)
## 16、函数参数
**参数定义**
在python中定义函数的时候，函数名后面的括号里就是用来定义参数的，如果有多个参数的话，那么参数之间直接用逗号, 隔开。**定义函数的时候在函数名后面的括号里定义的参数叫做形参，而调用函数的时候传入的参数叫做实参，形参是用来接收实参的**。

**参数分类**
1.位置参数
位置参数也称必备参数，是必须按照正确的顺序传到函数中，即调用时的数量和位置必须和定义时是一样的。

```python
def func(a,b):
    print(a)
    print(b)
     
add_num(11,22)
#运行结果
11
22
```

2.关键字参数
关键字参数是指使用形式参数的名字来确定输入的参数值。通过该方式指定实际参数时，不再需要与形式参数的位置完全一致。只要将参数名写正确即可。

```python
def func(a,b,c):
    print(a)
    print(b)
    print(c)
     
add_num(11,c=99,b=33)
#运行结果
11
33
99
```

**注意**：在定义函数时，指定默认的形式参数必须在所有参数的最后，否则将产生语法错误。
**可变参数**
在Python中，还可以定义可变参数。可变参数也称为不定长参数，即传入函数中的实际参数可以是任意多个。定义可变参数时，主要有两种形式：一种是 *parameter，另一种是 **parameter。
1. *parameter：这种形式表示接收任意多个实际参数并将其放到一个元组中。如果想要使用一个已经存在的列表作为函数的可变参数，可以在列表的名称前加 “ * ”。
2. **parameter：这种形式表示接收任意多个类似于关键字参数一样显式赋值的实际参数，并将其放到一个字典中。如果想要使用一个已经存在的字典作为函数的可变参数，可以在字典的名称前加 “ ** ”。
3. 在函数的形参中，如果同时有*parameter和\**parameter，*parameter必须在\**parameter前面。

```python
def func(*args):
　　print(args)
 
func(33,44,55,66,77)
func(*(33,44,55,66,77))
 
#运行结果
(33,44,55,66,77)
(33,44,55,66,77)
```

```python
def func(**kwargs):
    print(kwargs)
func(e=33,h=44,f=55,d=66,c=77)
func(**{'e':33,'h':44,'d':66,'c':77})
#运行结果
{'e': 33, 'h': 44, 'f': 55, 'd': 66, 'c': 77}
{'e': 33, 'h': 44, 'f': 55, 'd': 66, 'c': 77}
```
## 17、面向对象
**类和对象的定义**
类是一种用户自定义的数据类型，它由数据和方法组成。数据表示属性，方法表示行为。一个类可以包含多个属性和方法。属性是类的成员变量，可以存储数据。方法是一组操作数据的代码，它们可以实现某些功能或者改变属性的值。

对象是类的实例。通过类创建的对象具有类定义的所有属性和方法。

```python
class 类名:
    类属性 = 值
    def __init__(self, 参数):
        self.属性1 = 参数1
        self.属性2 = 参数2
    def 方法1(self, 参数):
        # 方法代码
    def 方法2(self, 参数):
        # 方法代码
 
__init__ 方法是一个特殊的方法，称为构造函数，用于初始化新创建的对象。
self 参数代表类的实例，必须作为第一个参数传递给每个实例方法。
```
**继承**
继承允许一个类（子类）继承另一个类（父类）的属性和方法。子类可以重写或扩展父类的方法。
**多态**
多态是指不同类的对象对同一消息作出不同的响应。在 Python 中，多态通常通过方法重写实现。
**封装**
封装是指将数据和操作数据的方法绑定在一起，并隐藏对象的内部实现细节。通过封装，可以保护对象的状态不被外部直接修改。

```bash
class BankAccount:
    def __init__(self, balance=0):
        self.__balance = balance  # 私有属性

    def deposit(self, amount):
        self.__balance += amount

    def withdraw(self, amount):
        if amount <= self.__balance:
            self.__balance -= amount
        else:
            print("余额不足")

    def get_balance(self):
        return self.__balance
```

**类中的私有函数与私有变量** 
1、什么是私有函数私有变量
无法被实例化后的对象调用的类中的函数与变量
类内部可以调用私有函数与变量
只希望类内部业务调用使用，不希望被使用者调用
2、私有函数与私有变量的定义
在变量或函数前添加__(2个下横线)，变量或函数名后面无需添加

Python中定义函数时，若想在函数内部对函数外的变量进行操作，就需要在函数内部将其声明其为global 变量。添加了global关键字后，则可以在函数内部对函数外的对象进行操作了，也可以改变它的值。

## 18、装饰器

1、重试装饰器

在数据科学项目和软件开发项目中，有很多我们依赖外部系统的情况。事情并不总是在我们的控制之中。当意外事件发生时，我们可能希望我们的代码等待一段时间，让外部系统自行纠正并重新运行。

```python
import time
from functools import wraps
def retry(max_tries=3, delay_seconds=1):
    def decorator_retry(func):
        @wraps(func)
        def wrapper_retry(*args, **kwargs):
            tries = 0
            while tries < max_tries:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    tries += 1
                    if tries == max_tries:
                        raise e
                    time.sleep(delay_seconds)
        return wrapper_retry
    return decorator_retry
@retry(max_tries=5, delay_seconds=2)
def call_dummy_api():
    response = requests.get("https://qzkq.github.io/")
    return response
print(call_dummy_api)
```
在上面的代码中，我们尝试获取 API 响应。如果失败，我们将重试相同的任务 5 次。在每次重试之间，我们等待 2 秒。

2、@logger

```python
def logger(function):
    def wrapper(*args, **kwargs):
        print(f"----- {function.__name__}: start -----")
        output = function(*args, **kwargs)
        print(f"----- {function.__name__}: end -----")
        return output
    return wrapper
    
@logger
def some_function(text):
    print(text)

some_function("first test")
# ----- some_function: start -----
# first test
# ----- some_function: end -----

some_function("second test")
# ----- some_function: start -----
# second test
# ----- some_function: end -----
```
3、@timeit
该装饰器用来测量函数的执行时间并打印出来。这对调试和监控非常有用。

```python
import time
from functools import wraps

def timeit(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f'{func.__name__} took {end - start:.6f} seconds to complete')
        return result
    return wrapper

@timeit
def process_data():
    time.sleep(1)

process_data()
# process_data took 1.000012 seconds to complete
```
4、@property
@property装饰器用于定义类属性，这些属性本质上是类实例属性的getter、setter和deleter方法。通过使用@property装饰器，可以将方法定义为类属性，并将其作为类属性进行访问，而无需显式调用该方法。如果您想在获取或设置值时添加一些约束和验证逻辑，使用@property装饰器会非常方便。

```python
class Movie:
    def __init__(self, r):
        self._rating = r

    @property
    def rating(self):
        return self._rating

    @rating.setter
    def rating(self, r):
        if 0 <= r <= 5:
            self._rating = r
        else:
            raise ValueError("The movie rating must be between 0 and 5!")

batman = Movie(2.5)
batman.rating
# 2.5

batman.rating = 4
batman.rating
# 4

batman.rating = 10
# ---------------------------------------------------------------------------
# ValueError                                Traceback (most recent call last)
# Input In [16], in <cell line: 1>()
# ----> 1 batman.rating = 10
# Input In [11], in Movie.rating(self, r)
#      12     self._rating = r
#      13 else:
# ---> 14     raise ValueError("The movie rating must be between 0 and 5!")
#
# ValueError: The movie rating must be between 0 and 5!
```
## 19、python 类的高级函数
1.__str__
__str__的功能，如果定义了该函数，当print当前实例化对象的时候，会返回该函数的return信息。

```python
class Test(object）：
    def __str__(self):
        return '这是描述'
 
test = Test()
print(test)
```
2.__getattr__
功能：当调用的属性或者方法不存在时，会返回该方法定义的信息。

```python
class Test2(object):
    def __getattr__(self,key):
        print('这个key:{}不存在'.format(key))
 
a = Test2()
print(a.w)
```
3.__setattr__
功能：拦截当前类中不存在的属性和值。

```python
def __setattr__(self, key, value):
    self.__dict__[key] = value
    print(self.__dict__)
```

4.__call__
功能：将一个类变成一个函数。

```python
class Test3(object):
 
    def __call__(self, a):
        print(a)
 
t = Test3()
t('测试')
#打印结果   测试
```
综合示例（魔法函数）

```python
# t.a.b.c 链式操作
class Test2(object):
    def __init__(self, attr=''):
        self.__attr = attr
 
    def __call__(self, name):
        return name
 
    def __getattr__(self, key):
        if self.__attr:
            key = '{}.{}'.format(self.__attr, key)
        else:
            key = key
        print(key)
        return Test2(key)
 
t2 = Test2()
name = t2.a.b.c('dewei')
print(name)
 
result = t2.name.age.sex('ok')
print(result)
```

> a
a.b
a.b.c
dewei
name
name.age
name.age.sex
ok

## 20、捕获异常(通用)

```python
try:
    <代码块>
except Exception as e:
    <异常代码块>
```
 **主动抛出异常**
可以使用 raise 语句来抛出异常，该语句后面需要带一个对象，该对象必须是派生自 Exception。

```python
nums = [1, 2, 3, 4]

try:
    num = 5
    if num in nums:
        print(num)
    else:
        myException = Exception("变量不在列表内...")	# 创建一个异常对象
        raise myException	# 主动抛出异常
except Exception as err:	# 接受异常，err的内容就是错误原因
    print(err)	# 输出异常信息（或者针对异常做其他处理）
```
**自定义异常类**

 1. 一定要继承Exception类
 2. 要重新定义 init 和 __str__函数

```python
nums = [1, 2, 3, 4]

# 继承异常基类 Exception
class myError(Exception):

    # 下面两个魔法函数是必须要写的

    # __init__函数负责类变量的初始化(一般是接报错的内容)
    def __init__(self, message):
        self.message = message

    # __str__函数负责根据类对象名称，返回异常信息
    def __str__(self):
        return "出现错误：" + self.message

try:
    num = 5
    if num in nums:
        print(num)
    else:
        myerror = myError("数字不在列表内")  # 创建一个自定义异常类型的变量
        raise myerror  # 手动抛出异常
except Exception as err:
    print(err)
```

**常见异常**

```python
BaseException 所有异常的基类
Exception 常见错误的基类
ArithmeticError 所有数值计算错误的基类
Warning 警告的基类

AssertError 断言语句（assert）失败
AttributeError 尝试访问未知的对象属性
DeprecattionWarning 关于被弃用的特征的警告
EOFError 用户输入文件末尾标志EOF（Ctrl+d）
FloattingPointError 浮点计算错误
FutureWarning 关于构造将来语义会有改变的警告
GeneratorExit generator.close()方法被调用的时候
ImportError 导入模块失败的时候
IndexError 索引超出序列的范围
KeyError 字典中查找一个不存在的关键字
KeyboardInterrupt 用户输入中断键（Ctrl+c）
MemoryError 内存溢出（可通过删除对象释放内存）
NamerError 尝试访问一个不存在的变量
NotImplementedError 尚未实现的方法
OSError 操作系统产生的异常（例如打开一个不存在的文件）
OverflowError 数值运算超出最大限制
OverflowWarning 旧的关于自动提升为长整型（long）的警告
PendingDeprecationWarning 关于特征会被遗弃的警告
ReferenceError 弱引用（weak reference）试图访问一个已经被垃圾回收机制回收了的对象
RuntimeError 一般的运行时错误
RuntimeWarning 可疑的运行行为（runtime behavior）的警告
StopIteration 迭代器没有更多的值
SyntaxError Python的语法错误
SyntaxWarning 可疑的语法的警告
IndentationError 缩进错误
TabError Tab和空格混合使用
SystemError Python编译器系统错误
SystemExit Python编译器进程被关闭
TypeError 不同类型间的无效操作
UnboundLocalError 访问一个未初始化的本地变量（NameError的子类）
UnicodeError Unicode相关的错误（ValueError的子类）
UnicodeEncodeError Unicode编码时的错误（UnicodeError的子类）
UnicodeDecodeError Unicode解码时的错误（UnicodeError的子类）
UserWarning 用户代码生成的警告
ValueError 传入无效的参数
ZeroDivisionError 除数为零
```
**assert断言**

```python
assert condition, message
```

 - condition：一个布尔表达式，表示你要断言的条件。如果条件为假，就会触发断言异常。
 - message：可选参数，通常是一个字符串，用于在触发断言异常时提供额外的信息，帮助你理解断言失败的原因
## 21、Python dir() 函数
dir() 函数不带参数时，返回当前范围内的变量、方法和定义的类型列表；带参数时，返回参数的属性、方法列表。如果参数包含方法__dir__()，该方法将被调用。如果参数不包含__dir__()，该方法将最大限度地收集参数信息。
