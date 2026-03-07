---
title: Tkinter详解和爱心跳动示例（Python）
date: 2026-01-03 08:12:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Tkinter详解和爱心跳动示例（Python）)
# Tkinter简要介绍

Tkinter 是 Python 的标准 GUI（图形用户界面）库。它提供了创建窗口和对话框的基本工具，可以用来构建各种复杂的用户界面。

## 1.基本概念

- Tk: Tk 是 Tkinter 的底层实现，最初是为 Tcl 语言设计的。Tkinter 是 Tk 的 Python 接口。
- Widget: Tkinter 中的控件，如按钮、标签、文本框等。
- Event: 用户与界面的交互，如点击按钮、键盘输入等。
- Callback: 事件触发后执行的函数。

### 1.1Tk

在 Tkinter 中，Tk 是一个类，用于创建主窗口。以下是关于 Tk 类的一些关键点：

- 创建主窗口：通过实例化 Tk 类来创建应用程序的主窗口。
- 初始化方法：通常使用 `__init__` 方法进行初始化，但大多数情况下直接调用 Tk() 即可。
- 进入事件循环：使用 mainloop() 方法启动事件循环，使窗口保持打开状态并响应用户操作。
- 窗口标题：可以使用 title() 方法设置窗口的标题。
- 窗口大小：可以通过 geometry() 方法设置窗口的初始大小和位置。

```python
import tkinter as tk

# 创建主窗口
root = tk.Tk()

# 设置窗口标题
root.title("我的应用")

# 设置窗口大小
root.geometry("400x300")

# 进入事件循环
root.mainloop()
```

### 1.2 Canvas
在 Tkinter 中，Canvas 是一个非常强大的小部件，用于绘制图形、文本、图像等。


**创建 Canvas**：通过实例化 Canvas 类来创建一个画布。

**常用方法**：

 - create_line(x1, y1, x2, y2, ...)：绘制线条。
 - create_rectangle(x1, y1, x2, y2, ...)：绘制矩形。
 - create_oval(x1, y1, x2, y2, ...)：绘制椭圆。
 - create_polygon(x1, y1, x2, y2, ..., xn, yn, ...)：绘制多边形。
 - create_text(x, y, text="")：绘制文本。
 - create_image(x, y, image=image_object, anchor=NW)：绘制图像。

**配置选项：**

 - width 和 height：设置画布的宽度和高度。
 - bg：设置背景颜色。

**绑定事件**：可以使用 bind 方法为画布上的对象绑定事件。

```python
import tkinter as tk

# 创建主窗口
root = tk.Tk()
root.title("Canvas 示例")

# 创建 Canvas 小部件
canvas = tk.Canvas(root, width=400, height=300, bg='white')
canvas.pack()

# 绘制线条
canvas.create_line(50, 50, 350, 50, fill='blue')

# 绘制矩形
canvas.create_rectangle(50, 100, 150, 200, fill='green')

# 绘制椭圆
canvas.create_oval(200, 100, 300, 200, fill='red')

# 绘制文本
canvas.create_text(200, 250, text="Hello, Tkinter!", fill='black')

# 进入事件循环
root.mainloop()
```

在 Tkinter 的 Canvas 小部件中，delete 方法用于删除画布上的特定项目或所有项目。

 - 删除特定项目：通过传递项目的标识符（ID）来删除特定的项目。
 - 删除所有项目：通过传递特殊标签 ALL 来删除画布上的所有项目。
 - 项目标识符：每个项目在创建时都会返回一个唯一的标识符，可以用来引用该项目。

**常用方法**

 - canvas.delete(item_id)：删除指定标识符的项目。
 - canvas.delete(tk.ALL)：删除画布上的所有项目。

```python
import tkinter as tk

# 创建主窗口
root = tk.Tk()
root.title("Canvas 删除示例")

# 创建 Canvas 小部件
canvas = tk.Canvas(root, width=400, height=300, bg='white')
canvas.pack()

# 绘制一些项目
line_id = canvas.create_line(50, 50, 350, 50, fill='blue')
rect_id = canvas.create_rectangle(50, 100, 150, 200, fill='green')
oval_id = canvas.create_oval(200, 100, 300, 200, fill='red')

# 删除特定项目
def delete_specific_item():
    canvas.delete(line_id)  # 删除线条

# 删除所有项目
def delete_all_items():
    canvas.delete(tk.ALL)  # 删除所有项目

# 创建按钮
btn_delete_specific = tk.Button(root, text="删除线条", command=delete_specific_item)
btn_delete_specific.pack()

btn_delete_all = tk.Button(root, text="删除所有项目", command=delete_all_items)
btn_delete_all.pack()

# 进入事件循环
root.mainloop()
```
### 1.3 pack
在 Tkinter 中，pack 是一种几何管理器（geometry manager），用于将小部件（widgets）放置到其父容器中。pack 管理器通过将小部件“打包”到父容器的边缘来自动排列它们。

**基本用法**：通过调用小部件的 pack() 方法将其添加到父容器中。

**常用选项**：

 - side：指定小部件在父容器中的对齐方式，可选值有 TOP（默认）、BOTTOM、LEFT 和 RIGHT。
 - fill：指定小部件是否填充父容器的剩余空间，可选值有 NONE（默认）、X、Y 和 BOTH。
 - expand：指定小部件是否扩展以填充父容器的额外空间，布尔值 True 或 False。
 - padx 和 pady：指定小部件与父容器之间的外部填充（外边距）。
 - ipadx 和 ipady：指定小部件内部的填充（内边距）。

```python
import tkinter as tk

# 创建主窗口
root = tk.Tk()
root.title("Pack 示例")

# 创建多个小部件
button1 = tk.Button(root, text="按钮1")
button2 = tk.Button(root, text="按钮2")
button3 = tk.Button(root, text="按钮3")

# 使用 pack 方法布局小部件
button1.pack(side=tk.TOP, fill=tk.X, padx=10, pady=5)
button2.pack(side=tk.LEFT, fill=tk.Y, ipadx=10, ipady=5)
button3.pack(side=tk.RIGHT, fill=tk.Y, ipadx=10, ipady=5)

# 进入事件循环
root.mainloop()
```

### 1.4 main.after
在 Tkinter 中，main.after 是一个非常有用的方法，用于在指定的时间间隔后执行某个函数或方法。它常用于实现定时任务、动画效果或延迟操作。

**基本用法**：main.after(ms, function, *args)，其中 ms 是时间间隔（毫秒），

 - function 是要执行的函数，*args 是传递给函数的参数。
 - 递归调用：可以通过在函数内部再次调用 after 方法来实现周期性的任务。\
 - 取消任务：可以使用 after_cancel 方法取消尚未执行的任务。

**常用方法**

 - main.after(ms, function, *args)：在 ms 毫秒后调用 function 函数，并传递 *args 参数。
 - main.after_cancel(id)：取消由 after 方法返回的任务标识符 id 所表示的任务。
## 2.常用控件
### 2.1 Label
在 Tkinter 中，Label 是一个用于显示文本或图像的小部件。它通常用于在界面上显示静态信息。

**创建 Label**：通过实例化 Label 类来创建一个标签。

**常用选项**：

 - text：设置标签显示的文本。
 - image：设置标签显示的图像。
 - font：设置文本的字体样式。
 - fg 或 foreground：设置文本的颜色。
 - bg 或 background：设置背景颜色。
 - width 和 height：设置标签的宽度和高度（单位为字符或像素）。
 - anchor：设置文本在标签中的对齐方式（例如 N、S、E、W、CENTER 等）。
 - justify：设置多行文本的对齐方式（例如 LEFT、CENTER、RIGHT）。

**方法**：

 - config()：用于修改标签的属性。
 - pack(), grid(), place()：用于布局标签。

#### 显示文本

```python
import tkinter as tk

# 创建主窗口
root = tk.Tk()
root.title("Label 示例")

# 创建一个 Label 小部件
label = tk.Label(root, text="欢迎使用 Tkinter！", font=("Arial", 16), fg="blue", bg="white")
label.pack(pady=20)

# 进入事件循环
root.mainloop()
```
#### 显示图像

```python
import tkinter as tk
from PIL import Image, ImageTk

# 创建主窗口
root = tk.Tk()
root.title("Label 图像示例")

# 加载图像
image = Image.open("path/to/your/image.png")
photo = ImageTk.PhotoImage(image)

# 创建一个 Label 小部件显示图像
label = tk.Label(root, image=photo)
label.pack(pady=20)

# 进入事件循环
root.mainloop()
```

#### 修改标签属性

```python
import tkinter as tk

# 创建主窗口
root = tk.Tk()
root.title("Label 属性修改示例")

# 创建一个 Label 小部件
label = tk.Label(root, text="初始文本", font=("Arial", 16), fg="blue", bg="white")
label.pack(pady=20)

# 修改标签的文本
def change_text():
    label.config(text="文本已更改", fg="red")

# 创建一个按钮，点击时修改标签的文本
button = tk.Button(root, text="更改文本", command=change_text)
button.pack(pady=10)

# 进入事件循环
root.mainloop()
```

### 2.2 Text
Text 小部件是一个多行文本编辑器，可以用来显示和编辑多行文本。它支持插入、删除和格式化文本，还可以插入图像和嵌入其他小部件。

**常用方法和属性**

 1.创建 Text 小部件：

```python
text_box = Text(root, wrap=WORD, bg='black', fg='white', font=('Arial', 12), height=5, width=20)
```

 2. 放置 Text 小部件：

```python
text_box.place(x=10, y=10)
```

 3. 插入文本：

```python
text_box.insert(END, "Your text here\n")
```

 4. 删除文本：
```python
text_box.delete('1.0', END)  # 删除所有文本
text_box.delete('1.0', '1.5')  # 删除从第1行第1个字符到第1行第5个字符之间的文本
```

 5. 获取文本：

```python
content = text_box.get('1.0', END)  # 获取所有文本
```

 6. 配置属性：

```python
text_box.config(state=DISABLED)  # 禁用文本编辑
text_box.config(state=NORMAL)  # 启用文本编辑
```

# 跳动的爱心

```python
import random
from math import sin, cos, pi, log
from tkinter import *

CANVAS_WIDTH = 640  # 画布的宽
CANVAS_HEIGHT = 480  # 画布的高
CANVAS_CENTER_X = CANVAS_WIDTH / 2  # 画布中心的X轴坐标
CANVAS_CENTER_Y = CANVAS_HEIGHT / 2  # 画布中心的Y轴坐标
IMAGE_ENLARGE = 11  # 放大比例
HEART_COLOR = "#FF69B4"  # 心的颜色
STAR_COLOR = "white"  # 星星的颜色


# 生成随机星星的位置和大小
def generate_stars(num_stars):
    stars = []
    for _ in range(num_stars):
        x = random.randint(0, CANVAS_WIDTH)
        y = random.randint(0, CANVAS_HEIGHT)
        size = random.randint(1, 3)
        stars.append((x, y, size))
    return stars


def heart_function(t, shrink_ratio: float = IMAGE_ENLARGE):
    """
    “爱心函数生成器”
    :param shrink_ratio: 放大比例
    :param t: 参数
    :return: 坐标
    """
    # 基础函数
    x = 16 * (sin(t) ** 3)
    y = -(13 * cos(t) - 5 * cos(2 * t) - 2 * cos(3 * t) - cos(4 * t))

    # 放大
    x *= shrink_ratio
    y *= shrink_ratio

    # 移到画布中央
    x += CANVAS_CENTER_X
    y += CANVAS_CENTER_Y

    return int(x), int(y)


def scatter_inside(x, y, beta=0.15):
    """
    随机内部扩散
    :param x: 原x
    :param y: 原y
    :param beta: 强度
    :return: 新坐标
    """
    ratio_x = - beta * log(random.random())
    ratio_y = - beta * log(random.random())

    dx = ratio_x * (x - CANVAS_CENTER_X)
    dy = ratio_y * (y - CANVAS_CENTER_Y)

    return x - dx, y - dy


def shrink(x, y, ratio):
    """
    抖动
    :param x: 原x
    :param y: 原y
    :param ratio: 比例
    :return: 新坐标
    """
    force = -1 / (((x - CANVAS_CENTER_X) ** 2 + (y - CANVAS_CENTER_Y) ** 2) ** 0.6)  # 这个参数...
    dx = ratio * force * (x - CANVAS_CENTER_X)
    dy = ratio * force * (y - CANVAS_CENTER_Y)
    return x - dx, y - dy


def curve(p):
    """
    自定义曲线函数，调整跳动周期
    :param p: 参数
    :return: 正弦
    """
    # 可以尝试换其他的动态函数，达到更有力量的效果（贝塞尔？）
    return 2 * (2 * sin(4 * p)) / (2 * pi)


class Heart:
    """
    爱心类
    """

    def __init__(self, generate_frame=20):
        self._points = set()  # 原始爱心坐标集合
        self._edge_diffusion_points = set()  # 边缘扩散效果点坐标集合
        self._center_diffusion_points = set()  # 中心扩散效果点坐标集合
        self.all_points = {}  # 每帧动态点坐标
        self.build(2000)

        self.random_halo = 1000

        self.generate_frame = generate_frame
        for frame in range(generate_frame):
            self.calc(frame)

    def build(self, number):
        # 爱心
        for _ in range(number):
            t = random.uniform(0, 2 * pi)  # 随机不到的地方造成爱心有缺口
            x, y = heart_function(t)
            self._points.add((x, y))

        # 爱心内扩散
        for _x, _y in list(self._points):
            for _ in range(3):
                x, y = scatter_inside(_x, _y, 0.05)
                self._edge_diffusion_points.add((x, y))

        # 爱心内再次扩散
        point_list = list(self._points)
        for _ in range(6000):
            x, y = random.choice(point_list)
            x, y = scatter_inside(x, y, 0.17)
            self._center_diffusion_points.add((x, y))

    @staticmethod
    def calc_position(x, y, ratio):
        # 调整缩放比例
        force = 1 / (((x - CANVAS_CENTER_X) ** 2 + (y - CANVAS_CENTER_Y) ** 2) ** 0.520)  # 魔法参数

        dx = ratio * force * (x - CANVAS_CENTER_X) + random.randint(-1, 1)
        dy = ratio * force * (y - CANVAS_CENTER_Y) + random.randint(-1, 1)

        return x - dx, y - dy

    def calc(self, generate_frame):
        ratio = 10 * curve(generate_frame / 10 * pi)  # 圆滑的周期的缩放比例

        halo_radius = int(4 + 6 * (1 + curve(generate_frame / 10 * pi)))
        halo_number = int(3000 + 4000 * abs(curve(generate_frame / 10 * pi) ** 2))

        all_points = []

        # 光环
        heart_halo_point = set()  # 光环的点坐标集合
        for _ in range(halo_number):
            t = random.uniform(0, 4 * pi)  # 随机不到的地方造成爱心有缺口
            x, y = heart_function(t, shrink_ratio=11.5)  # 魔法参数
            x, y = shrink(x, y, halo_radius)
            if (x, y) not in heart_halo_point:
                # 处理新的点
                heart_halo_point.add((x, y))
                x += random.randint(-14, 14)
                y += random.randint(-14, 14)
                size = random.choice((1, 2, 2))
                all_points.append((x, y, size))

        # 轮廓
        for x, y in self._points:
            x, y = self.calc_position(x, y, ratio)
            size = random.randint(1, 3)
            all_points.append((x, y, size))

        # 内容
        for x, y in self._edge_diffusion_points:
            x, y = self.calc_position(x, y, ratio)
            size = random.randint(1, 2)
            all_points.append((x, y, size))

        for x, y in self._center_diffusion_points:
            x, y = self.calc_position(x, y, ratio)
            size = random.randint(1, 2)
            all_points.append((x, y, size))

        self.all_points[generate_frame] = all_points

    def render(self, render_canvas, render_frame):
        for x, y, size in self.all_points[render_frame % self.generate_frame]:
            render_canvas.create_rectangle(x, y, x + size, y + size, width=0, fill=HEART_COLOR)


def draw(main: Tk, render_canvas: Canvas, render_heart: Heart, render_frame=0):
    render_canvas.delete('all')

    # 绘制星星
    stars = generate_stars(100)  # 生成100颗星星
    for x, y, size in stars:
        render_canvas.create_rectangle(x, y, x + size, y + size, width=0, fill=STAR_COLOR)

    render_heart.render(render_canvas, render_frame)
    main.after(160, draw, main, render_canvas, render_heart, render_frame + 1)


def change_label_color(label, colors, index=0):
    label.config(fg=colors[index])
    index = (index + 1) % len(colors)
    label.after(500, change_label_color, label, colors, index)


if __name__ == '__main__':
    root = Tk()
    root.title('Beating_heart')

    # 创建画布
    canvas = Canvas(root, bg='black', height=CANVAS_HEIGHT, width=CANVAS_WIDTH)
    canvas.pack()

    # 创建左上角的文本框
    # text_box = Text(root, wrap=WORD, bg='black', fg='white', font=('Arial', 12), height=5, width=20)
    # text_box.place(x=10, y=10)

    # 插入社交网址
    # social_urls = [
    #     "ZhiHu: https://www.zhihu.com/people/qin-zheng-kai-89",
    #     "Csdn: https://blog.csdn.net/qq_45832050?type=blog",
    #     "GitHub: https://github.com/QInzhengk",
    # ]
    # for url in social_urls:
    #     text_box.insert(END, url + '\n')

    heart = Heart()
    draw(root, canvas, heart)

    # 创建Label并设置初始颜色
    colors = ["#FF69B4", "#FFD700", "#00FF00", "#00BFFF", "#FF1493"]
    label = Label(root, text="peace and love", bg="black")
    label.place(relx=.5, rely=.5, anchor=CENTER)

    # 启动颜色变化
    change_label_color(label, colors)

    root.mainloop()

```
![在这里插入图片描述](/img/c3c7f3dd99f34e839e28dfc23db11631.png)

## 参考

1.[Graphical User Interfaces with Tk — Python 3.13.0 documentation](https://docs.python.org/3/library/tk.html)

2.[Python - GUI 编程 (tutorialspoint.com)](https://www.tutorialspoint.com/python/python_gui_programming.htm)

3.[https://github.com/star-start/Beating_heart](https://github.com/star-start/Beating_heart)




