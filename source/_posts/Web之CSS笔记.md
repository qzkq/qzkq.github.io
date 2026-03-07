---
title: Web之CSS笔记
date: 2026-01-03 08:14:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Web之HTML、CSS、JS)

**[Web之HTML笔记](https://blog.csdn.net/qq_45832050/article/details/134346218)**
**[Web之JavaScript(jQuery)笔记](https://blog.csdn.net/qq_45832050/article/details/134481763)**
### 二、CSS（Cascading Style Sheets层叠样式表）
Css是种格式化网页的标准方式， 用于控制设置网页的样式，并且允许CSS样式信息与网页内容(由HTML语言定义)分离的一种技术。

**优势**

 - 格式和结构分离：有利于格式的重用以及网页的修改与维护。 
 - 精确控制页面布局：对网页实现更加精确的控制，如网页的布局，字体，颜色，背景等。
 - 实现多个网页同时更新。

#### CSS与HTML的结合方式
**方式一：内联/行内样式**
在HTML标签上通过style属性来引用CSS代码。

 - 优点：简单方便 
 - 缺点：只能对一个标签进行修饰

```css
<body>
    <div style="color: blue">聚沙成塔，滴水穿石。</div>
</body>
```

**方式二：内部样式表**
通过`<style>`标签来声明CSS。

 - 优点：可以通过多个标签进行统一的样式设置
 - 缺点：只能在本页面上进行修饰

**语法： 选择器 {属性:值; 属性:值}**

```css
<style>
    div {
        color: blueviolet;
    }
</style>

<body>
    <div>聚沙成塔，滴水穿石。</div>
</body>
```

**方式三：外部样式表**
单独定义一个CSS文件，CSS文件的后缀名就是.css。
在`<head>`中使用`<link>`标签引用外部的css文件

```css
/* test.css */
div {
    color: blueviolet
}
<!DOCTYPE html>
<html lang="en">   <!-- 英文 -->

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Hello, HTML!</title>
    <link rel="stylesheet" href="test.css">
</head>

<body>
    <div>聚沙成塔，滴水穿石。</div>
</body>

</html>
```

**`<meta>`标签**

属性|	描述
|--|--|
charset	|规定 HTML 文档的字符编码。
content	|定义与 http-equiv 或 name 属性相关的元信息。
http-equiv|	把 content 属性关联到 HTTP 头部。
name	|把 content 属性关联到一个名称。
scheme|	定义用于翻译 content 属性值的格式。

#### CSS选择器
**元素（标签）选择器**
它可以对页面上相同的标签进行统一的设置，它描述的就是标签的名称。

```css
<style>
    div {
        color: cyan
    }
</style>

<body>
    
    <div>聚沙成塔，滴水穿石。</div>
    <div>千里之行，始于足下。</div>
</body>
```

**类选择器 & id选择器**
类选择器在使用时使用 . 来描述，它描述的是元素上的class属性值。
id选择器只能选择一个元素，使用 # 引入，引用的是元素的id属性值。（比类选择器更具唯一性）

```css
<style>
    .a {
        color: cyan
    }

    #b {
        color: blue
    }
</style>

<body>
    <div class="a">聚沙成塔，滴水穿石。</div>
    <div id="b">千里之行，始于足下。</div>
</body>
```

#### CSS基本属性
**背景属性**

 - background-color 设置元素的背景颜色。 
 - background-image 把图像设置为背景。
 - background-repeat 设置背景图像的墙纸效果，是否及如何重复 repeat：在垂直方向和水平方向重复 repeat-x：仅在水平方向重复 repeat-y：仅在垂直方向重复 no-repeat：仅显示一次
 - background-position 设置背景图像的起始位置 
 - background-attachment 背景图像固定或随着页面滚动 默认值是 scroll：默认情况下，背景会随文档滚动 可取值为fixed：背景图像固定，并不会随着页面的其余部分滚动，常用于实现称为水印的图像

```css
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
		p{
			background-color: red;
			font-size: 40px;
		}
		.p1{
			font-family: 隶书;
		}
		body{
			/*background-color:yellow;
			background-image: url("image/wudaojiaoshi.jpg");
			background-repeat: no-repeat;
			background-attachment: fixed;
			background-position: 20px 30px;*/
			background:yellow url("image/wudaojiaoshi.jpg") no-repeat fixed 30px 40px;
		}
	</style>
</head>
<body>
	<p>http://www.baidu.com</p>
	<p class="p1">baidu</p>
	<p class="p1">百度</p>
</body>
</html>
```

**文本属性**

 - 指定字体：`font-family` : value
 - 字体大小：`font-size` : value
	 - 		px：像素  
	 - 		em：倍数 
 - 字体加粗：`font-weight` : normal / bold
 - 规定斜体文本： `font-style` : italic倾斜，强调实现斜体字 / oblique倾斜，更注重实现倾斜效果（常用）/   normal正常显示（默认值）
 - 文本颜色：`color` : value
 - 文本方向：`dircetion` : ltr, rtl
 - 字符间距：`letter-spacing` : npx(n可以是负数)
 - 行高：`line-height` : value
 - 文本排列：`text-align` : left / right / center/ ustify:两端对齐(应用在多行文本中，单行文本不生效)
 - 文字修饰：`text-decoration` : none / underline / line-through（删除线）/ overline
 - 文本设置阴影：`text-shadow`
 - 首行文本缩进：`text-indent` : value（nem, npx）
 - 大小写字母转换：`text-transform` : uppercase(全部大写) / lowercase(全部小写) / capitalize(在所有小写单词中，首字母大写)
 - 列表属性 类型：`list-style-type` : disc(圆点) / circle(圆圈) / square(方块) / decimal(数字)… 位置：list-style-position : outside(外) / inside
 - 图像：`list-style-image` : url(…)

#### CSS伪类
用于已有元素处于某种状态时（滑动、点击等）为其添加对应的样式，这个状态是根据用户行为而动态变化的。

动态伪类|	作用
|--|--
:link|	链接没有被访问前的样式效果
:visited|	链接被访问后的样式效果
:hover|	鼠标悬停在元素上面时的样式效果
:active|	点击元素时的样式效果，即按下鼠标左键时发生的样式
:focus|	用于元素成为焦点时的样式效果，常用与表单元素

```css
a:link{
	color:red;
}
a:visited{
	color: green;
}
a:hover{
	color: yellow;
	font-size: 30px;
}
a:active{
	color:blue;
}
label:hover{
	color:red;
}
input:hover{
	background-color: red;
}
input:active{
	background-color: blue;
}
input:focus{
	background-color: yellow;
}
```

结构伪类	|作用
|--|--|
:first-child|	选择某个元素的第一个子元素
:last-child|	选择某个元素的最后一个子元素
:nth-child()|	选择某个当前元素的兄弟节点下的一个或多个特定的子元素
:nth-last-child()|	选择某个当前元素的兄弟节点的一个或多个特定的子元素，从后往前数
:first-of-type|	选择一个父级元素下第一个同类型子元素

伪元素选择器	|作用|
|--|--|
::selection|	选择指定元素中被用户选中的内容
::before|	可以在内容之前插入新内容
::after|	可以在内容之后插入新内容
::first-line|	选择指定选择器的首行
::first-letter|	选择文本的第一个字符

```css
/*可以将p换成h1*/
p::before{
	content: "终于找到你，";
}
/*可以将body也换成h1*/
body::after{
	content: "依依不舍离开你，";
}
p::first-line{
	background-color: yellow;
}
p::first-letter{
	font-size: 30px;
}
p::selection{
	background-color: red;
}
```

#### DIV
DIV是层叠样式表中的定位技术，全称DIVision；有时把div称为图层，更多时候称为“块”。

**DIV溢出处理效果**

 - 超出div宽度高度的文字或图片进行隐藏处理
 - 超出div宽度高度的文字或图片增加滚动条

overflow属性|	描述
|--|--|
visible|	默认值。内容不会被修剪，会呈现在元素框之外。
hidden|	内容会被修剪，并且其余内容是不可见的。
scroll|	内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。
auto	|如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。
inherit|	规定应该从父元素继承 overflow 属性的值。

#### CSS轮廓
CSS轮廓是用于在元素周围绘制线条的属性，位于边框边缘的外围，可以起到突出元素的作用。轮廓的样式、颜色和宽度可以通过CSS outline属性进行规定，轮廓是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用。

轮廓属性	|说明
|--|--|
outline-color|	设置轮廓的颜色
outline-style|	设置轮廓的样式 solid（实线）、dotted（点线）、dashed（虚线）、double（双线）、groove（凹槽）、ridge（垄脊）、inset（内嵌）、outset（外凸）或 none（无轮廓）
outline-width|	设置轮廓的宽度
outline-offset|	设置轮廓与元素边框之间的距离 像素值或正值百分比

#### CSS边框
指定元素边框的样式、宽度和颜色。

border属性|	说明
|--|--|
border-width|	指定边框的宽度
border-style|	指定边框的样式
border-color|	指定边框的颜色

```css
#div1{
	background-color: yellow;
	width: 150px;
	height: 150px;
	top:150px;
	left:150px;
	position: absolute;
	overflow: hidden;
	/*outline: none;*/
	border-bottom: solid;
}

#div2{
	top:150px;
	left:350px;
	position: absolute;
	/*border-bottom: solid;*/
}

input{
	border:none;
	border-bottom: solid;
	outline: none;
}
```

#### 盒子模型
CSS 中规定每个盒子分别由： 内容区域（content）、内边距区域（padding）、边框区域（border）、外边距区域（margin） 构成，这就是 盒子模型。
![在这里插入图片描述](/img/9d64b14146df85b0ddaebebb5e6fe7c2.png)


```css
div{
	width: 200px;
	height: 200px;
	overflow: hidden;
	margin-left: 20px;
}
#div1{
	background-color: yellow;
	margin-top: 20px;
	margin-bottom: 20px;
	padding-right: 20px;
	box-sizing: border-box;
}
#div2{
	background-color: blue;

}

*{
	/*margin:0px 0px 0px 0px;*/
	/*margin:0px 0px;*/
	margin-top: 0px;
	margin-left: 0px;
	margin-bottom: 0px;
	margin-right:0px;
}
```

**行级元素与块级元素的区别**

 - 行级元素：行内元素和其他行内元素都会在一条水平线 上排列，都是在同一行的；a标签、label、img、span等
 - 块级元素：块级元素在默认情况下，会独占一行；div 、h标签、li、table等

#### CSS定位
CSS position属性用于指定一个元素在文档中的定位方式，定位分为静态定位，相对定位，绝对定位，固定定位、粘性定位这五种；其中top、right、bottom、left和z-index属性则决定了该元素的最终位置。

