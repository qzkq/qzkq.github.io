---
title: Web之HTML笔记
date: 2026-01-03 08:15:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Web之HTML、CSS、JS)

**[Web之CSS笔记](https://blog.csdn.net/qq_45832050/article/details/134480138)**
**[Web之JavaScript(jQuery)笔记](https://blog.csdn.net/qq_45832050/article/details/134481763)**
# Web标准

 - 结构标准用于对网页元素进行整理和分类(**HTML**) 
 - 表现标准用于设置网页元素的版式、颜色、大小等外观属性(**CSS**) 
 - 行为标准用于对网页模型的定义及交互的编写(**JavaScript**)

## 一、HTML（超文本标记语言）
### HTML 基本结构标签
|标签名| 含义 |说明|
|--|--|--|
|`<html></html>`  |HTML标签  |页面中的最大的标签，称为跟标签|
|`<head></head>`  |文档的头部  |注意在head标签中必须要设置的标签是title|
|`<title></title>`  |文档的标题  |让页面拥有一个属于自己的网页标题|
|`<body></body>`  |文档的主体  |元素包含文档的所有内容，页面内容基本都是放到body里面的|

### 常用标签
#### 1.font标签 
部分常用属性（html5不支持该标签，建议使用CSS）：

    color属性：修改颜色
    face属性：修改字体（类型）
    size属性：修改文本大小（1-7）

```html
<font color="red" face="黑体" size="3">font字体标签</font>
```
<font color="red" face="黑体" size="3">font字体标签</font>
#### 2.p标签
全称paragraph，用来表示段落，它是一个行内元素，一个标签独占一行。

```html
<p>......</p>
```
<p>......</p>

```html
<br>：全称barter rabbet,换行标签，用于插入一个换行符。
```
<br>

#### 3.注释

```html
<!-- 需求： 在网页上显示 font字体标签 ， 并修改字体为 黑体， 颜色为红色。-->
```
<!-- 需求： 在网页上显示 font字体标签 ， 并修改字体为 黑体， 颜色为红色。-->
#### 4.h系列标题
`<h1>`到`<h6>`：标题标签，用于定义标题的级别，`<h1>`是最高级别的标题，依次递减。
```html
<h1>标题标签1</h1>
<h2>标题标签2</h2>
<h3>标题标签3</h3>
<h4>标题标签4</h4>
<h5>标题标签5</h5>
<h6>标题标签6</h6>
```
<h1>标题标签1</h1>
<h2>标题标签2</h2>
<h3>标题标签3</h3>
<h4>标题标签4</h4>
<h5>标题标签5</h5>
<h6>标题标签6</h6>

#### 5.img
`<img>`: 图像标签，用于插入图像；通过src属性指定图像的URL，可以是相对路径或绝对路径。

```html
<img src="test.jpg" alt="风景" title="夜景" height="100" />
```

常用属性
  
     alt属性：alt属性用于指定图像的替代文本。当图像无法加载时，替代文本会显示在图像的位置。
     width和height属性：width和height属性用于指定图像的宽度和高度。可以使用像素(px)、百分比(%)或其他单位来指定。
     title:提示文本,鼠标放到图片上,就会有提示

#### 6.超链接a
用于从一个页面链接到另一个页面。

```html
<a href="https://bilibili.com">网址直接跳转</a> 
<a href="images/1.webp">相对路径跳转</a>
<a href="hello.exe">如果是打不开的文件，则下载之</a>
<a href="">空链接是刷新</a>
<a href="#">#是回到顶部</a>
<a href="javascript:;">禁止链接跳转</a>
```
<a href="https://bilibili.com">网址直接跳转</a> 
<a href="images/1.webp">相对路径跳转</a>
<a href="hello.exe">如果是打不开的文件，则下载之</a>
<a href="">空链接是刷新</a>
<a href="#">#是回到顶部</a>
<a href="javascript:;">禁止链接跳转</a>

常用属性

	target:打开方式,默认是_self.如果是_blank则用新的标签页打开

**锚点**

href里面为#id变为锚点功能，点击跳转到id对应的块。

1.快速定位到页面中的某个位置。

```html
<a href="#one">第一集</a>
<a href="#two">第二集</a>
<a href="#three">第三集</a>
<p id="one">
   第一集剧情 <br>
   第一集剧情 <br>
   ...
</p>
<p id="two">
   第二集剧情 <br>
   第二集剧情 <br>
 ...
</p>
<p id="three">
   第三集剧情 <br>
   第三集剧情 <br>
 ...
</p>
```
<a href="#one">第一集</a>
<a href="#two">第二集</a>
<a href="#three">第三集</a>
<p id="one">
   第一集剧情 <br>
   第一集剧情 <br>
   ...
</p>
<p id="two">
   第二集剧情 <br>
   第二集剧情 <br>
 ...
</p>
<p id="three">
   第三集剧情 <br>
   第三集剧情 <br>
 ...
</p>

2.跳转到不同页面的不同位置

```html
<a href=”demo.html#锚点名称”>demo.html页面 xxx元素位置</a>

<a href=”demo.html#box”>demo.html页面box元素位置</a>
```
<a href=”demo.html#锚点名称”>demo.html页面 xxx元素位置</a>

<a href=”demo.html#box”>demo.html页面box元素位置</a>

#### 7.列表
**ul ——无序列表**

```html
 <ul>
        <li>无序列表1</li>
        <li>无序列表2</li>
        <li>无序列表3</li>
        <li>无序列表4</li>
 </ul>
```

 <ul>
        <li>无序列表1</li>
        <li>无序列表2</li>
        <li>无序列表3</li>
        <li>无序列表4</li>
    </ul>

**ol ——有序列表**

```html
  <ol>
        <li>有序列表1</li>
        <li>有序列表2</li>
        <li>有序列表3</li>
        <li>有序列表4</li>
    </ol>
```

  <ol>
        <li>有序列表1</li>
        <li>有序列表2</li>
        <li>有序列表3</li>
        <li>有序列表4</li>
    </ol>

#### 8.表格
一种用于展示结构化数据的标记语言元素。
表格由 `<table>` 标签来定义。每个表格均有若干行（由 `<tr>` 标签定义），每行被分割为若干单元格（由 `<td>` 标签定义）。字母 td 指表格数据（table data），即数据单元格的内容。数据单元格可以包含文本、图片、列表、段落、表单、水平线、表格等。

	caption标签:表格的标题
	thead标签:表格的页眉
	tbody标签:表格的主体
	tfoot标签:表格的页脚
	th标签:行/列的标题，文字加粗显示
```html
<table border="1px" bgcolor="green" bordercolor="yellow" width="300px"
		height="175px">
			<caption>鲜鱼价目表</caption>
			<thead><!-- 表头部分 -->
				<tr>
					<th>序号</th>
					<th>鱼的种类</th>
					<th>价格</th>
				</tr>
			</thead>
			<tbody> <!--表主体部分-->
				<tr align="center">
					<td>1</td>
					<td>草鱼</td>
					<td>18.6</td>
				</tr>
				<tr valign="top">
					<td>2</td>
					<td>鲤鱼</td>
					<td>28.9</td>
				</tr>
				<tr>
					<td>3</td>
					<td>食人鱼</td>
					<td>1000</td>
				</tr>
			</tbody>
		</table>
```
<table border="1px" bgcolor="green" bordercolor="yellow" width="300px"
		height="175px">
			<caption>鲜鱼价目表</caption>
			<thead><!-- 表头部分 -->
				<tr>
					<th>序号</th>
					<th>鱼的种类</th>
					<th>价格</th>
				</tr>
			</thead>
			<tbody> <!--表主体部分-->
				<tr align="center">
					<td>1</td>
					<td>草鱼</td>
					<td>18.6</td>
				</tr>
				<tr valign="top">
					<td>2</td>
					<td>鲤鱼</td>
					<td>28.9</td>
				</tr>
				<tr>
					<td>3</td>
					<td>食人鱼</td>
					<td>1000</td>
				</tr>
			</tbody>
		</table>
		
**table标签的属性**

 - border="1px"  设置边框 
 - bgcolor="green"  设置背景颜色 
 - bordercolor="yellow"  设置边框颜色 
 - width="300px" 设置表格的宽度 
 - height="175px"  设置表格的高度
 - table表格里的边框是带有间距的 解决方案就是给table标签加: `style="border-collapse: collapse;"` 去掉边框间的间距 
 - align="center" 设置表格本身的水平对齐方式，注意不是文字居中，而是整张表格在页面居中显示

**tr标签的属性**

 - align="" 设置内容的水平对齐方式 left靠左/center居中/right靠右 
 - valign="" 设置内容的垂直对齐方式 top靠上/middle居中/bottom靠下

**td标签的属性**

 - colspan="n" 跨列，从当前单元格开始，向右合并n个单元格(n也包含当前单元格) 
 - rowspan="n" 跨行，从当前单元格开始，向下合并n个单元格(n也包含当前单元格)

```html
<!-- table>tr*3>td*4 在数字后按Tab补全-->
	<table border="1px" width="300px" height="200px">
		<tr>
			<td colspan="2">1-1</td>
			<!-- 被合并的单元格一定得删掉 -->
			<!-- <td>1-2</td> -->
			<td>1-3</td>
			<td>1-4</td>
		</tr>
		<tr>
			<td>2-1</td>
			<td rowspan="2">2-2</td>
			<td>2-3</td>
			<td>2-4</td>
		</tr>
		<tr>
			<td>3-1</td>
			<!-- 被合并的单元格一定要删掉！ -->
			<!-- <td>3-2</td> -->
			<td>3-3</td>
			<td>3-4</td>
		</tr>
	</table>
	<hr> <!-- 创建一条水平线 -->
```

<!-- table>tr*3>td*4 在数字后按Tab补全-->
<table border="1px" width="300px" height="200px">
	<tr>
		<td colspan="2">1-1</td>
		<!-- 被合并的单元格一定得删掉 -->
		<!-- <td>1-2</td> -->
		<td>1-3</td>
		<td>1-4</td>
	</tr>
	<tr>
		<td>2-1</td>
		<td rowspan="2">2-2</td>
		<td>2-3</td>
		<td>2-4</td>
	</tr>
	<tr>
		<td>3-1</td>
		<!-- 被合并的单元格一定要删掉！ -->
		<!-- <td>3-2</td> -->
		<td>3-3</td>
		<td>3-4</td>
	</tr>
</table>
<hr> 

#### 9.表单
HTML 表单用于收集用户的输入信息。
HTML 表单表示文档中的一个区域，此区域包含交互控件，将用户收集到的信息发送到 Web 服务器。
一个完整的表单包含三个基本组成部分：表单标签、表单域、表单按钮。

```html
<form>表单标签  
	<!-- 表单域包含了文本框、密码框、隐藏域、多行文本框、复选框、单选框、下拉选择框和文件上传框等 -->
    <input type="text">表单域 
    <button type="submit">提交按钮</button >
</form>
```
**input属性**

 - type

|type值	|表单类型|
|--|--|
|text|	单行文本框|
passworld|	密码文本框
button	|按钮
reset	|重置按钮
image|	图像形式的提交按钮
radio|	单选按钮
checkbox|	复选框
hidden	|隐藏字段
file	|文件上传
 - name属性：给出当前input表单的名字。
 - value属性：表示该input表单的默认值。
1）当input type=“text”、“password”、"hidden"时，value中的值会成为其输入框的初始值；
2）当input type=“button”、“reset”、"submit"时，定义按钮上的显示的文本；
3）当input type=“checkbox”、“radio”、"image"时，定义与输入相关联的值；
注意：input type="checkbox"和input type="radio"中必须设置
 - value属性；value属性无法与input type="file"一通使用。
 - style属性：为input元素设置简单的CSS样式。
 - width属性：当input type="image"时，通过width属性控制元素的宽度；
 - height属性：当input type="image"时，通过height属性控制元素的高度；
 - maxlength属性：定义input元素中可输入的最长字符数。

**select和option创建下拉式表单**

```html
<select>      
    <option value="1" selected="selected">qq.com</option>
    <option value="2">163.com</option>
    <option value="3">tongji.edu.cn</option>
</select>
```
selected标注默认选中的内容。

**textarea标签创立多行文本框**

```html
<textarea name="introduction" id="introduction" cols="30" rows="10"></textarea>
```

**表单示例**
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>表单</title>
</head>
<body>
	<form action="" method="">
		<label>请输入姓名：</label>
		   <input type="text" name="" id=""><br>
		<label>请输入密码：</label>
			<input type="password" name="" id=""><br>
		<label>再次输入密码：</label>
			<input type="password" name="" id=""><br>
		<lebel>性别：</lebel>
			<input type="radio" name="xb" id="" value="1">男
			<input type="radio" name="xb" id="" value="0">女<br>
		<label>兴趣爱好</label>
			<input type="checkbox" name="" id="" value="1">游泳
			<input type="checkbox" name="" id="" value="2">看书
			<input type="checkbox" name="" id="" value="3">爬山
			<input type="checkbox" name="" id="" value="4">思考<br>
		<label>生日：</label>
			<select>
				<option value="1995">1995</option>
				<option value="1996">1996</option>
				<option value="1997" selected="selected">1997</option>
				<option value="1998">1998</option>
				<option value="1999">1999</option>
				<option value="2000">2000</option>
			</select>年
			<select>
				<option value="1">01</option>
				<option value="2">02</option>
				<option value="3">03</option>
				<option value="4">4</option>
				<option value="5">5</option>
			</select>月
			<select>
				<option value="1">01</option>
				<option value="2">02</option>
				<option value="3">03</option>
				<option value="4">4</option>
				<option value="5">5</option>
			</select>日<br>
			头像<img src="image/headLogo/13.gif">
			<select>
				<option value="1">1</option>
				<option value="2">2</option>
				<option value="3">3</option>
				<option value="4">4</option>
			</select><br>
			<input type="button" value="普通按钮">
			<input type="submit" value="提交按钮">
	</form>
	<textarea rows="10" cols="100" name="" id="">
	  请输入
	</textarea>
	<input type="file"><input type="button" value="上传"><br>
	000<input type="hidden" name="" id="">000
	       <select size="4" multiple="true">
				<option value="1">1</option>
				<option value="2">2</option>
				<option value="3">3</option>
				<option value="4">4</option>
				<option value="4">41</option>
				<option value="42">42</option>
				<option value="43">43</option>
				<option value="44">44</option>
				<option value="45">45</option>
			</select>

			 <select size="4" multiple="true">
			 </select>
</body>
</html>
```
![在这里插入图片描述](/img/1d822ee4f42a8ac8a485ad1fb0ccbd2c.png)
