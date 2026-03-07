---
title: 笔记-pd.set_option()、warnings、np.set_printoptions参数详解
date: 2026-01-05 08:11:00
tags:
  - "Python"
  - "Pandas"
  - "pd.set_option"
  - "warnings"
  - "笔记"
categories:
  - "数据科学"
---

﻿[ 微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&amp;mid=100000290&amp;idx=1&amp;sn=96634a7b9e474b976599aa5d931af97f&amp;scene=19&token=1993860133&lang=zh_CN#wechat_redirect)
## 一、pd.set_option()
pd.set_option() 函数用于设置 pandas 的各种显示选项。
```python
import pandas as pd

# 设置显示所有列
pd.set_option('display.max_columns', None)

# 设置显示所有行
pd.set_option('display.max_rows', None)

# 设置显示宽度为 100 字符
pd.set_option('display.width', 100)

# 设置浮点数的小数位数为 2
pd.set_option('display.precision', 2)

# 设置浮点数的格式化字符串
pd.set_option('display.float_format', '{:.2f}'.format)

# 设置大数据集的显示方式为 'info'
pd.set_option('display.large_repr', 'info')

# 不换行显示宽表格
pd.set_option('display.expand_frame_repr', False)

# 设置最大列宽为 100 字符
pd.set_option('display.max_colwidth', 100)

# 显示 DataFrame 的维度信息
pd.set_option('display.show_dimensions', True)

# 使用 HTML 表格模式
pd.set_option('display.html.table_schema', True)

# 在 Jupyter Notebook 中使用 HTML 表格
pd.set_option('display.notebook_repr_html', True)

# 设置序列的最大显示项数为 100
pd.set_option('display.max_seq_items', 100)

# 设置小数值的截断阈值为 1e-10
pd.set_option('display.chop_threshold', 1e-10)

# 示例 DataFrame
df = pd.DataFrame({
    'A': [1, 2, 3, 4, 5],
    'B': [1.123456789, 2.123456789, 3.123456789, 4.123456789, 5.123456789],
    'C': ['a' * 100, 'b' * 100, 'c' * 100, 'd' * 100, 'e' * 100]
})

print(df)
```

## 二、warnings
在 Python 编程中，warnings 模块被用来发出警告信息，这些警告通常用来指出可能的问题，但又不足以抛出异常终止程序执行的情况。警告可以用来提醒开发者注意某些潜在的问题，比如过时的函数使用、未来可能会改变的行为等。

warnings 模块定义了多个警告类别，例如：

```python
UserWarning：用于通用的用户级别的警告。
DeprecationWarning：用于警告某个特性即将被废弃。
PendingDeprecationWarning：用于警告某个特性未来可能会被废弃。
SyntaxWarning：用于警告可疑的语法。
RuntimeWarning：用于警告可疑的运行时行为。
ImportWarning：用于警告导入时可能出现的问题。
FutureWarning：用于警告将来可能会改变的行为。
RuntimeError：用于警告资源可能有问题。
```

```python
import warnings
warnings.filterwarnings('ignore')  # 关闭运行时的警告
# 忽略所有的 UserWarning
warnings.filterwarnings("ignore", category=UserWarning)
```
## 三、np.set_printoptions
np.set_printoptions 是 NumPy 库中的一个函数，用于设置打印数组时的显示选项。通过这个函数，你可以控制 NumPy 数组在打印时的行为，例如小数点后的位数、数组的显示宽度等。

### 常用参数
**precision**

作用：设置浮点数的小数点后位数。

默认值：8

**threshold**

作用：设置完整打印数组的元素数量阈值。如果数组的元素数量超过这个阈值，则会以省略号（...）的形式简化打印。

默认值：sys.getoption("numpy.printthreshold")，通常是 1000

**edgeitems**

作用：设置当数组被简化打印时，每端显示的元素数量。

默认值：3

**linewidth**

作用：设置每行的最大宽度（字符数）。

默认值：80

**suppress**

作用：设置是否抑制小数点后多余的零。

默认值：False

**nanstr**

作用：设置 NaN（Not a Number）的显示字符串。

默认值：'nan'

**infstr**

作用：设置无穷大（Infinity）的显示字符串。

默认值：'inf'

**formatter**

作用：设置自定义的格式化函数字典，可以针对不同的数据类型设置格式化函数。

默认值：None

```python
import numpy as np

# 设置打印选项
np.set_printoptions(precision=4, suppress=True, linewidth=100, threshold=5, edgeitems=2)

# 创建一个数组
arr = np.random.rand(10)

# 打印数组
print(arr)
```
![在这里插入图片描述](/img/d382747f7a634a1cbde58fe0ca25aa8f.png)

