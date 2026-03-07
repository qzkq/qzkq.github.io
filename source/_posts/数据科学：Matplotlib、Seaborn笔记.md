---
title: 数据科学：Matplotlib、Seaborn笔记
date: 2026-01-04 08:10:00
tags:
  - "数据科学"
  - "可视化"
  - "Matplotlib"
  - "Seaborn"
  - "笔记"
categories:
  - "数据科学"
---

﻿[**数据科学：Numpy、Pandas**](https://blog.csdn.net/qq_45832050/article/details/127466841)

[**数据科学：Matplotlib、Seaborn笔记**](https://blog.csdn.net/qq_45832050/article/details/134764886)
@[TOC](数据科学：Numpy、Pandas、Matplotlib、Seaborn、Scipy、Scikit-Learn)
##  三、Matplotlib
### Figure的组成

1. Figure
顶层级：整个图表的容器，可以包含一个或多个 Axes 对象。
功能：设置图表的整体属性，如大小、分辨率、背景颜色等。
方法：如 add_subplot、add_axes 用于添加子图，savefig 用于保存图表等。
2. Axes
核心层级：容纳了大量元素，用于构造一幅幅子图。
功能：包含一个或多个 Axis 对象，处理绘图区域的所有元素，如线条、标记、文本等。
方法：如 plot、scatter、bar 用于绘制数据，set_xlabel、set_ylabel 用于设置轴标签等。
属性：可以设置坐标轴范围、标签、标题等。
3. Axis
下属层级：Axes 的子层级，用于处理所有与坐标轴和网格有关的元素。
功能：管理坐标轴的范围、刻度、标签、网格线等。
方法：如 set_xlim、set_ylim 设置坐标轴范围，grid 设置网格线等。
4. Tick
下属层级：Axis 的子层级，用于处理所有与刻度有关的元素。
功能：管理刻度的位置、标签、格式等。
方法：如 set_ticks 设置刻度位置，set_ticklabels 设置刻度标签，set_major_locator 设置主刻度定位器等。

### 两种绘图接口

1. 面向对象接口（Object-Oriented Interface）
面向对象接口提供了更灵活和可定制的绘图方式。通过这种方式，你可以明确地创建 Figure 和 Axes 对象，并直接在其上进行绘图操作。这种方式适用于需要精细控制图表外观和布局的复杂场景。


```bash
import matplotlib.pyplot as plt
import numpy as np

# 创建一个Figure对象和一个Axes对象
fig, ax = plt.subplots(figsize=(8, 6))

# 在Axes对象上绘制数据
x = np.linspace(0, 10, 100)
y = np.sin(x)
ax.plot(x, y, label='sin(x)')

# 设置标题和标签
ax.set_title('Sine Wave')
ax.set_xlabel('x')
ax.set_ylabel('sin(x)')

# 添加图例
ax.legend()

# 显示图表
plt.show()

```
2. 隐式接口（Pyplot Interface）
隐式接口是 Matplotlib 的传统接口，基于 MATLAB 的绘图风格。通过这种方式，你可以使用 pyplot 模块中的函数来创建和操作图表，而不需要显式地创建 Figure 和 Axes 对象。这种方式适用于快速绘制简单的图表。

```bash
import matplotlib.pyplot as plt
import numpy as np

# 创建数据
x = np.linspace(0, 10, 100)
y = np.sin(x)

# 使用pyplot接口绘制数据
plt.plot(x, y, label='sin(x)')

# 设置标题和标签
plt.title('Sine Wave')
plt.xlabel('x')
plt.ylabel('sin(x)')

# 添加图例
plt.legend()

# 显示图表
plt.show()

```

### 1.Matplotlib subplots函数

```python
ig, ax = plt.subplots(nrows=1, ncols=1, sharex=False, sharey=False, **kwargs)
```

 - nrows: 行数
 - ncols: 列数
 - sharex: 是否共享x轴刻度
 - sharey: 是否共享y轴刻度
 - figsize: 图形大小

```python
# 绘制多个箱线图
import matplotlib.pyplot as plt
import numpy as np

data1 = np.random.randn(1000)
data2 = np.random.randn(1000)

fig, axs = plt.subplots(2, sharex=True, sharey=True)
fig.suptitle('Multiple Boxplot')

axs[0].boxplot(data1)
axs[0].set_title('Boxplot 1')
axs[1].boxplot(data2)
axs[1].set_title('Boxplot 2')

plt.show()
```
### 2.tight_layout()函数
matplotlib库的pyplot模块中的tight_layout()函数用于自动调整子图参数以提供指定的填充。

```python
matplotlib.pyplot.tight_layout(pad=1.08, h_pad=None, w_pad=None, rect=None)
```

 - pad:此参数用于在图形边和子图的边之间进行填充，以字体大小的一部分表示。
 - h_pad，w_pad：这些参数用于相邻子图的边之间的填充(高度/宽度)，作为字体大小的一部分。
 - rect:此参数是整个子图区域将适合的归一化图形坐标中的矩形。

### 3.Matplotlib grid()设置网格格式
通过Matplotlib axes 对象提供的 grid() 方法可以开启或者关闭画布中的网格（即是否显示网格）以及网格的主/次刻度。除此之外，grid() 函数还可以设置网格的颜色、线型以及线宽等属性。

```python
grid(color='b', ls = '-.', lw = 0.25)
```

 - color：表示网格线的颜色；
 - ls：表示网格线的样式；
 - lw：表示网格线的宽度；

### 4.fill_between()函数
fill_between和fill_betweenx函数的作用都是填充两条曲线之间的区域。其中

 - fill_between函数作用是填充两条水平曲线之间的区域。
 - fill_betweenx函数作用是填充两条垂直曲线之间的区域。

```python
matplotlib.pyplot.fill_between(x, y1, y2=0, where=None, interpolate=False, step=None, *, data=None, **kwargs)
```
参数说明如下：

 - x：定义两条曲线的节点的x坐标。长度为N的类数组结构。必备参数。
 - y1：定义曲线的节点的y坐标。长度为N的类数组结构或者标量。必备参数。
 - y2：定义第2条曲线的节点的y坐标。长度为N的类数组结构或者标量，默认值为0。可选参数。
 - where：根据条件排除一些填充区域。长度为N的布尔数组。默认值为None。可选参数。
 - interpolate：当该属性为True时将计算实际交点，并将填充区域延伸到此点。布尔值。默认值为False。注意：该属性只有使用了where属性且两条曲线相互交叉时才生效。

### 5.add_subplot
add_subplot 是 Matplotlib 中的一个方法，用于在当前图形中添加一个子图。它可以在一个图形窗口中创建多个子图，并且支持二维和三维绘图。
函数签名

```python
figure.add_subplot(nrows, ncols, index, **kwargs)
```
参数:

 - nrows: 子图的行数。
 - ncols: 子图的列数。
 - index: 当前子图的索引（从 1 开始）。
 - projection: 指定子图的类型，例如 '3d' 表示三维子图。

### 6.plot_surface
plot_surface 是 mpl_toolkits.mplot3d.Axes3D 类中的一个方法，用于绘制三维表面图。它需要三个二维数组，分别表示 x、y 和 z 坐标的值。
**函数签名**

```python
Axes3D.plot_surface(X, Y, Z, *args, **kwargs)
```
参数:

 - X: 二维数组，表示 x 坐标。
 - Y: 二维数组，表示 y 坐标。
 - Z: 二维数组，表示 z 坐标。
 - rstride: 行步长，默认为 10。
 - cstride: 列步长，默认为 10。
 - cmap: 颜色映射，例如 'viridis'、'plasma' 等。
 - alpha: 透明度，取值范围为 0 到 1。

#### 示例

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

# 创建图形和轴
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

# 生成网格数据
x = np.linspace(-5, 5, 100)
y = np.linspace(-5, 5, 100)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

# 绘制曲面图
surface = ax.plot_surface(X, Y, Z, cmap='viridis', linewidth=0, antialiased=False)

# 添加颜色条
fig.colorbar(surface, ax=ax, shrink=0.5, aspect=5)

# 设置标签
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')

# 显示图形
plt.show()

参数详解
fig: 图形对象，通过 plt.figure() 创建。
ax: 三维轴对象，通过 fig.add_subplot(111, projection='3d') 创建。
X, Y, Z: 三维网格数据。X 和 Y 通常是通过 np.meshgrid 生成的网格坐标，Z 是对应的函数值。
cmap: 颜色映射，用于设置曲面的颜色。常用的有 'viridis', 'plasma', 'inferno', 'magma' 等。
linewidth: 曲面线的宽度。设置为 0 可以去掉线条。
antialiased: 是否启用抗锯齿。设置为 False 可以提高渲染速度。
```
通过修改 X, Y, Z 的生成方式来创建不同的三维曲面图。例如，绘制一个双峰函数的曲面图。

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

# 创建图形和轴
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

# 生成网格数据
x = np.linspace(-5, 5, 100)
y = np.linspace(-5, 5, 100)
X, Y = np.meshgrid(x, y)
Z = np.exp(-((X - 1)**2 + (Y - 1)**2)) + np.exp(-((X + 1)**2 + (Y + 1)**2))

# 绘制曲面图
surface = ax.plot_surface(X, Y, Z, cmap='coolwarm', linewidth=0, antialiased=False)

# 添加颜色条
fig.colorbar(surface, ax=ax, shrink=0.5, aspect=5)

# 设置标签
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')

# 显示图形
plt.show()
```

### 7.FuncAnimation 
matplotlib.animation 模块中的一个类，用于创建基于函数的动画。它允许你通过定义一个初始化函数和一个更新函数来创建动态的图表。
**基本语法**

```python
matplotlib.animation.FuncAnimation(fig, func, frames=None, init_func=None, fargs=None, save_count=None, **kwargs)
```
**参数说明**

 - fig: matplotlib.figure.Figure 对象，表示要绘制动画的图形。
 - func: 更新函数，每次调用时都会更新图形。函数签名应为 func(frame_number, *fargs)。
 - frames: 可选参数，指定动画的帧数或帧数据。可以是整数（表示帧数）、可迭代对象（表示每帧的数据）或生成器。
 - init_func: 可选参数，初始化函数，用于设置动画的初始状态。函数签名应为 init_func()。
 - fargs: 可选参数，传递给 func 和 init_func 的额外参数。
 - save_count: 可选参数，指定保存动画时保留的帧数。
 - **kwargs: 其他关键字参数，传递给 TimedAnimation 类。
#### 示例

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

# 创建图形和轴
fig, ax = plt.subplots()
x = np.linspace(0, 2 * np.pi, 100)
line, = ax.plot(x, np.sin(x))

# 初始化函数
def init():
    line.set_ydata([np.nan] * len(x))  # 初始状态下隐藏线条
    return line,

# 动画函数
def animate(i):
    line.set_ydata(np.sin(x + i / 10.0))  # 更新 y 数据
    return line,

# 创建动画对象
ani = FuncAnimation(fig, animate, init_func=init, frames=200, interval=20, blit=True)

# 显示动画
plt.show()

参数详解
fig: 图形对象，通过 plt.subplots() 创建。
animate: 更新函数，每次调用时更新图形。i 是当前帧的索引。
init: 初始化函数，设置动画的初始状态。
frames: 帧数或帧数据。这里设置为 200，表示动画有 200 帧。
interval: 每帧之间的时间间隔（毫秒）。这里设置为 20 毫秒，表示每帧间隔 20 毫秒。
blit: 是否使用双缓冲技术来提高性能。设置为 True 可以提高动画的流畅性。
关键点解释
初始化函数 (init): 用于设置动画的初始状态。在这个示例中，我们将线条的 y 数据设置为 np.nan，使其在初始状态下不可见。
更新函数 (animate): 每次调用时更新图形。在这个示例中，我们通过改变 x 的偏移量来更新正弦波的形状。
帧数 (frames): 动画的总帧数。在这个示例中，动画有 200 帧。
时间间隔 (interval): 每帧之间的时间间隔（毫秒）。在这个示例中，每帧间隔 20 毫秒。
双缓冲 (blit): 使用双缓冲技术可以提高动画的性能。设置为 True 可以减少重绘的开销。
```
通过修改 animate 函数来创建更复杂的动画效果。例如，添加多个线条、改变线条的颜色、添加文本标签等。

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

# 创建图形和轴
fig, ax = plt.subplots()
x = np.linspace(0, 2 * np.pi, 100)
line1, = ax.plot(x, np.sin(x), label='sin(x)')
line2, = ax.plot(x, np.cos(x), label='cos(x)')
ax.legend()

# 初始化函数
def init():
    line1.set_ydata([np.nan] * len(x))
    line2.set_ydata([np.nan] * len(x))
    return line1, line2

# 动画函数
def animate(i):
    line1.set_ydata(np.sin(x + i / 10.0))
    line2.set_ydata(np.cos(x + i / 10.0))
    return line1, line2

# 创建动画对象
ani = FuncAnimation(fig, animate, init_func=init, frames=200, interval=20, blit=True)

# 显示动画
plt.show()
```

### 设置x轴为时间刻度
```python
imoort pandas as pd
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

df = pd.read_excel("***.xlsx")
# 绘制图像
fig, ax = plt.subplots()
ax.plot(df['time'], df['*'])
# 配置x轴时间间隔
time_format = mdates.DateFormatter('%H:%M:%S')
ax.xaxis.set_major_formatter(time_format)
ax.xaxis.set_major_locator(mdates.MinuteLocator(interval=240))
# 设置刻度位置
ax.set_xticks(pd.date_range(df['time'][0], df['time'][-1], freq='4h'))
# 还可以使用ax.set_xticklabels()来设置刻度的标签
# 设置开始坐标
ax.set_xlim(df['time'][0], df['time'][-1])
# 旋转x轴标签
fig.autofmt_xdate()
# 展示图形
plt.show()
```
### 热力图
散点图坐标轴为数值型数据，热力图类别型数据，体现的是两组变量的相关性

```python
# 案例背景：工厂出货品质的好坏
factories = ['fac1','fac2','fac3','fac4','fac5']
quanlity = ['bad','poor','general','good','great']
result = np.round(np.random.random(25).reshape(5,5),1)

fig,ax = plt.subplots(1,1)

ax.imshow(result)

# 轮流锁定单元格
for i in np.arange(len(factories)):
    for j in np.arange(len(quanlity)):
        plt.text(j,i,result[i][j],color='w',ha='center',va='center')
        
# 设置坐标轴的类别数据标签
ax.set_xticks(np.arange(len(quanlity)))
ax.set_yticks(np.arange(len(factories)))
ax.set_xticklabels(quanlity)
ax.set_yticklabels(factories)

# 修饰工作
ax.set_title('goods quality of factories')
fig.tight_layout()
```
![在这里插入图片描述](/img/b6f11b136e3061ed4344cadf019a3ca2.png)
[**Python数据可视化matplotlib和pyecharts参数详解**](https://zhuanlan.zhihu.com/p/273263576)
## 四、Seaborn
### 1.set
设置绘图的背景色、风格、字型、字体等

```python
seaborn.set(context='notebook', style='darkgrid', palette='deep', font='sans-serif', font_scale=1, color_codes=True, rc=None)
```
Seaborn有五个预设好的主题： darkgrid， whitegrid，dark，white，和 ticks，默认为darkgrid

```python
控制风格：axes_style(), set_style()

缩放绘图：plotting_context(), set_context()
```

### 常用函数

```python
sns.distplot() – 绘制单变量分布图
sns.jointplot() – 绘制双变量关系图
sns.pairplot() – 绘制多变量关系图
sns.barplot() – 绘制条形图
sns.countplot() – 绘制计数图
sns.boxplot() – 绘制箱线图
sns.violinplot() – 绘制小提琴图
sns.heatmap() – 绘制热力图
sns.lineplot() – 绘制线图
sns.scatterplot() – 绘制散点图
```
### 3.seaborn.scatterplot

```python
seaborn.scatterplot(
    x=None, y=None,   - vectors or keys in data 作用：指定 x 轴和 y 轴上位置的变量。
    hue=None,         - vector or key in data 作用：将生成不同颜色的点的变量分组。
    					可以是分类或数字，尽管颜色映射在后一种情况下会有不同的行为。
    style=None,       - vector or key in data 作用：将生成具有不同标记的点的变量分组。
    					可以具有数字 dtype，但始终被视为分类类型。
    size=None,        - vector or key in data  作用：将生成不同大小的点的变量分组。
    					可以是分类型的，也可以是数值型的，尽管大小映射在后一种情况下会有不同的行为
    data=None,        - pandas.DataFrame, numpy.ndarray, mapping, or sequence 
    					作用：输入数据结构。要么是可以分配给命名变量的长形式向量集合，
    						要么是将在内部重新形成的宽形式数据集合。
    sizes=None,       - list, dict, or tuple  作用：一个对象，它决定使用时如何选择大小。
    					它始终可以是大小值的列表，或者是变量到大小的 dict 映射级别。
    					当是 numeric 时，它也可以是一个元组，指定要使用的最小和最大大小，
    					以便在这个范围内对其他值进行规范化
    size_order=None,  - list 作用：为变量级别的外观指定顺序，否则根据数据确定它们。
    					如果变量是 numeric.sizeize，则不相关
    size_norm=None,   - tuple or Normalize object 作用：当变量是 numeric.size 时，
    					用于缩放绘图对象的数据单元的规范化
    markers=True,     - boolean, list, or dictionary 
    					作用：对象，确定如何为变量的不同级别绘制标记。设置为将使用默认标记，
    						或者可以将变量的标记列表或字典映射级别传递给标记。
    						设置为将绘制无标记线。在 matplotlib.styleTruestyleFalse 中指定标记
    alpha=None,       - float 作用：点的比例不透明度。
): 
```


## 参考
1.[matplotlib中文](https://matplotlib.org.cn/)
2.[https://github.com/datawhalechina/fantastic-matplotlib](https://github.com/datawhalechina/fantastic-matplotlib)
3.[Matplotlib实践](https://tianchi.aliyun.com/course/324/?spm=a2c22.21852674.0.0.5abf5dcawliMZy)

