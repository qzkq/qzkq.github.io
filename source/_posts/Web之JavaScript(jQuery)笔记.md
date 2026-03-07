---
title: Web之JavaScript(jQuery)笔记
date: 2026-01-03 08:16:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿
@[TOC](Web之HTML、CSS、JavaScript)
**[Web之HTML笔记](https://blog.csdn.net/qq_45832050/article/details/134346218)
[Web之CSS笔记](https://blog.csdn.net/qq_45832050/article/details/134480138)**
### 三、JavaScript
JavaScript（简称“JS”）是一种轻量级的面向对象的编程语言，既能用在浏览器中控制页面交互，也能用在服务器端作为网站后台（借助 Node.js），因此 JavaScript 是一种全栈式的编程语言。

JavaScript 是一种跨平台的解释型语言，不需要提前编译，能在各种操作系统下运行。

页面使用js的方法：

 - 使用`<script>`在HTML页面中插入JavaScript

```html
<body>
    <script type="text/javascript">
        alert("hello javascript");
    </script>
</body>
```

 - 引用外部JS文件

```html
<script type="text/javascript" src="js1.js></script>
```
#### JS调试
**1.使用console输出**

```html
console.log('Hello World!');
```

console是一个非常便捷的调试工具，可以用来输出当前变量的值，也可以用来输出一些提示信息。

**2.alert**
alert是javascript中的一个内置函数，用于显示带有一条指定消息和一个“确认”按钮的警告框.

```html
alert("文本")
```

警告框经常用于确保用户可以得到某些信息；当警告框出现后，用户需要点击确定按钮才能继续进行操作。

**3. 使用Chrome开发者工具进行调试**
Chrome开发者工具是浏览器内置的一种调试工具，使用它可以进行变量查看、断点调试、性能分析等操作。例如：

	打开Chrome浏览器
	打开需要进行调试的网页
	在Chrome菜单栏中选择“开发者工具”
	在“Sources”面板中添加断点，然后运行代码
	调试过程中，可以通过“Console”面板查看变量的值，或者使用调试命令进行调试
	
Chrome开发者工具是比较常用的调试方法之一，可以方便地查看变量的值、调试执行路径、性能分析等操作，非常适合JavaScript编程中使用。

#### 变量
因为js是弱类型语言，所以，在定义变量的时候，所有的数据类型都是var。
声明变量：var x; var x,y;

**自动类型转换**

		数字 + 字符串：数字转换为字符串  10+'a'  -> 10a
		数字 + 布尔值：true转换为1，false转换为0   true+5 ->6
		字符串  +  布尔值：布尔值转换为字符串true 或 false  true+'a' -> truea
		布尔值  +  布尔值 ： 布尔值转换为数值1 或 0  true+true ->2

#### 自定义函数

1.在Javascript中必须用function关键字

```html
function functionName(parameters){
    //函数内的代码
    return value；
}
```
2.匿名函数

```html
var fucName = function(arg1, arg2, ...){
    statements;
}
```

```html
var num1=function(n1,n2){
	var n3=n1+n2;
	return n3;
}
var n=num1(14,14);
alert(n);
alert(num2(3,6));
function num2(n1,n2){
	return n1+n2;
}
```

#### 数据类型及转换
类型|作用
|--|--|
number|数字类型，整型浮点型都包括
srting|字符串类型，必须放在单引号或双引号中
boolean|布尔类型，只有true和false两种值
underfine|未定义，一般指的是已经声明，但是没有赋值的变量
null|空对象类型，var a = null, 和var a = ""有区别；
|**特殊类型**|**作用**
object|对象类型，在js常见的有window，document，array等
NaN|是Number的一种特殊类型，isNaN()，如果是数字返回false，不是数字返回true

**数据类型转换函数**

 - **parseInt：强制转换成整数**

如果不能转换，则返回NaN（NaN属性是代表非数字值的特殊值。）
例如：parseInt(“6.32”)=6

 - **parseFloat:强制转换成浮点数**

如果不能转换，则返回NaN
例如：parseFloat(“6.32”)=6.32

 - **Number() 转换数值**

 - Boolean() 转换布尔值

```html
// var str="123";
// console.log(str*1+1);
// console.log(parseInt(str)+1);

var str1="abc";
if(!isNaN(str1)){
	console.log(parseInt(str1));
}else{
	console.log("is error");
	str1=0;
}
console.log(str1);
```
#### 运算符优先级
![在这里插入图片描述](/img/41a63e1dc0ee00c2b653b14401fb0eee.png)
#### 内置函数
![在这里插入图片描述](/img/ad483951e4574e4249e0777a42f5345a.png)

 - substring(开始,结束)：截取字符串中一部分（结束是不包含的）
 - charAt(下标):返回某个下标上的字符
 - split(分割的节点):一个字符串切割成N个 小字符串，所以返回的是数组类型
 - length：获取字符串的长度（字符串中字符的个数）属性，没有小括号
 - indexof(字符):查找字符串中字符出现的首次下标
 - replace(旧的,新的):将字符串中的旧字符串替换成新字符
 - concat(新元素):将原来的数组连接新元素，原数组不变

```html
var arr = [1,2,3,4];
var arrnew = arr.concat(5,6);   //在arr数组的后面添加新的元素，形成一个新数组，但是原数组是不变的
console.log(arrnew +",类型为：" + typeof(arrnew));
console.log("原数组:" + arr);
```

```html
var d1=new Date();
var d2=new Date("2020-1-1");
console.log(d1.getDate());
console.log(d1.getMonth()+1);//从0开始，0-11
console.log(d1.getFullYear());
console.log(d1.getHours());
console.log(d1.getMinutes());
console.log(d1.getSeconds());
var n=d2.getTime()-d1.getTime();
console.log(parseInt(n/(24*3600*1000)));
//2020-1-1 15:58   日期格式化

function fun_FmtDate(){
  var d1=new Date();
  var yyyy,mm,dd,hh,mi,ss;
  var time;
  yyyy=d1.getFullYear()
  mm=d1.getMonth()+1;  //月份从0开始，11结束
  dd=d1.getDate();
  hh=d1.getHours();
  mi=d1.getMinutes();
  ss=d1.getSeconds();
  time=yyyy+"-"+mm+"-"+dd+" "+hh+":"+mi+":"+ss;
  return time;
}
console.log(fun_FmtDate());
```
#### 数组

```html
// 声明或创建一个不指定长度的数组，又称实例化创建：
// var arrayObj=new Array();
// // 声明或创建一个数组并指定长度的数组：　
// var arrayObj=new Array(5);
// // 声明或创建一个带有默认值的数组：
// var arrayObj=new Array(2,4,"a","y",8);
// // 创建一个数组并赋值的简写.又称隐式创建数据
// var arrayObj=[2,4,"a","y",8,5,1];
//数组赋值、字符下标、数组遍历
//console.log(arrayObj[5]);
// for(var i in arrayObj){
//    console.log(arrayObj[i]);
// }
var i=0;
var n=arrayObj.length;
for(i;i<n;i++){
   console.log(arrayObj[i]);
}
```
#### 事件
事件是指被程序发现的行为或发生的事情，而且它可能会被程序处理。
JS的事件，都是以on开头。
![在这里插入图片描述](/img/12c8387fc7eb61dc6bbf0d6028693aa5.png)
**表单元素事件（Form Element Events）**
仅在表单元素中有效

 - onblur 当元素失去焦点时执行脚本
 - onfocus 当元素获得焦点时执行脚本
 - onSubmit 当表单提交时触发
 - onChange ：当状态改变时触发，常用于select下拉选框

**键盘事件**

 - onkeydown 按下去 
 - onkeyup 弹上来
 - onkeypress ：当键盘按下时触发（要快于onkeydown）

**鼠标事件（Mouse Events）**

 - onclick 当鼠标被单击时执行脚本 
 - ondblick 当鼠标被双击时执行脚本 
 - onmouseout 当鼠标指针移出某元素执行脚本
 - onmouseover 当鼠标指针悬停于某元素之上时执行脚本
 - onMouseDown ：当鼠标按下时触发

#### DOM(Document Object Model 文档对象模型)
将文档（页面）表现为结构化的表示方法，使每一个页面元素都是可操控，DOM将页面和脚本以及其他的编程语言联系了起来。

**DOM树**
![在这里插入图片描述](/img/a7f3d39aca1a2726d3d3c5afd792108e.png)
核心 DOM：针对任何结构化文档的标准模型。 XML 和 HTML 通用的标准
 - Document：整个文档对象 
 - Element：元素对象 
 - Attribute：属性对象 
 - Text：文本对象 
 - Comment：注释对象

**获取 Element对象**
HTML 中的 Element 对象可以通过 Document 对象获取，而 Document 对象是通过 window 对象获取。

1.Document 对象中提供了以下获取 Element 元素对象的函数

 - `getElementById()`：根据id属性值获取，返回单个Element对象
 - `getElementsByTagName()`：根据标签名称获取，返回Element对象数组
 - `getElementsByName()`：根据name属性值获取，返回Element对象数组
 - `getElementsByClassName()`：根据class属性值获取，返回Element对象数组
 - document.querySelector(selector): 根据选择器获取第一个匹配的元素。
 - document.querySelectorAll(selector): 根据选择器获取所有匹配的元素。

2.创建元素：

 - document.createElement(tagName): 创建一个指定标签名的元素节点。
 - document.createTextNode(text): 创建一个包含指定文本内容的文本节点。

3.修改元素属性和内容：

 - element.setAttribute(name, value): 设置元素的属性值。
 - element.getAttribute(name): 获取元素的属性值。
 - element.innerHTML: 设置或获取元素的HTML内容。
 - element.innerText: 设置或获取元素的文本内容。

4.添加和删除元素：

 - parentElement.appendChild(newChild): 将一个新的子节点添加到指定父节点的子节点列表的末尾。
 - parentElement.removeChild(child): 从指定父节点的子节点列表中删除一个子节点。


[**Python读写xml（xml，lxml）**](https://blog.csdn.net/qq_45832050/article/details/131390858)

#### jQuery
jQuery是一个快速、简洁的JavaScript框架。 使用户能更方便地处理HTML, css, dom…

 - DOM对象：用原生JS获取过来的对象，一般使用原生的JS方法和属性
 - jQuery对象：通过$把DOM元素获取过来（以伪数组形式存储），不能使用DOM对象的原生JS方法和属性

**jQuery选择器**
$(“选择器”)要直接写css选择器，记住加引号

选择器 | 写法
|--|--|
ID选择器 | $("#id")
全选选择器| $("*") 匹配所有元素
类选择器| $(".class")
标签选择器| $(“标签”) eg：div
并集选择器| $(“div,p,li”) 选取多个元素
交集选择器| $(“li.current”)
子代选择器| $(“ul>li”) 只获取亲儿子层级的元素
后代选择器 |$(“ul li”) 中间空格表示，获取ul下边的所有li，包括孙子

**筛选选择器**

语法 |用法
|--|--|
:first |$(“li:first”) 获取第一个li元素
:last |$(“li:last”) 获取最后一个li元素
:eq(index)| $(“li:eq(2)”) 获取到的li元素中选取索引号是2的元素
:odd |$(“li:odd”) 获取li元素中，选取奇数
:even| $(“li:even”) 获取li元素中，选取偶数

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<title>jQuery选择器的练习</title>
	<link rel="stylesheet" href="">
	<script src="./js/jquery-3.4.1.js" type="text/javascript"></script>
	<script type="text/javascript">
		// jQuery代码的内容
		// $(document).ready(function() {
		// 	// 根据ID
		// 	var username = $('#username');
		// 	// var username = jQuery('#username');
		// 	console.log(username);

		// 	// 根据class来查询
		// 	var areaList = jQuery('.area');
		// 	console.log(areaList);

		// 	// 根据元素标签来查询
		// 	var pList = $('p');
		// 	console.log(pList)
		// });

		// $(function() {
		jQuery(function() {
			// 根据ID
			var username = $('#username');
			// var username = jQuery('#username');
			console.log(username);

			// 根据class来查询
			var areaList = jQuery('.area');
			console.log(areaList);

			// 根据元素标签来查询
			var pList = $('p');
			console.log(pList)

			// 层级选择器
			// var bodyList = $('body *');
			// 所有的后代元素p
			var bodyList = $('body p');
			console.log(bodyList);

			// 直接的子元素
			bodyList = $('body > p');
			console.log(bodyList);

			// 紧贴之后的元素
			var input = $('label + input');
			console.log(input)

			// 伪类选择器练习
			var pFirst = $('p:first');
			console.log(pFirst)

			var p2  = $('p:eq(1)');
			console.log(p2);

			// 属性选择器
			var password = $('input[name="password"]');
			// var password = $('input[name^="passw"]');
			console.log(password)

			var idInputList = $('input[id]');
			console.log(idInputList)
		})

	</script>
</head>
<body>
	<label for="username">用户名</label>
	<input type="text" name="username" id="username">
	<input type="password" name="password" >

	<p class="area city">广州</p>
	<p>深圳</p>
	<p class="area">长沙</p>
	<p class="area" id="beij">北京</p>

	<div>
		<p>海南</p>
	</div>
	
</body>
</html>
```

**jQuery属性操作**

 - element.attr(“属性”)获取元素的自定义属性
 - element.attr(“属性”，“属性值”)设置元素的自定义属性
 - val()获得表单元素中的value值 
 - val("x")修改表单元素中的value值 
 - html()获得元素中的内容（标签+文本） 
 - html("x")修改元素中的内容（标签+文本） 
 - text()获得元素中的文本 
 - text("x")修改元素中的文本


```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<title>DOM查询</title>
	<link rel="stylesheet" href="">
	<script src="./js/jquery-3.4.1.min.js" type="text/javascript"></script>
</head>
<body>
	<input type="text" name="username" id="username" class="input-text user-input" my-user='张三'
		value="我的用户名">
		<!-- textarea  select checkbox radio -->
	<input type="text" name="password">

	<p class="area city">广州</p>
	<p style="color:#f00;">深圳</p>
	<p class="area">长沙 <span>测试数据</span></p>
	<p class="area" id="beij">北京</p>

	<p class="info" id="info" style="display: none;">
		查看详细
		<span>内容：</span>
		<small>文字描述</small>
	</p>
	<script>
		$(function() {
			var pList = $('p');
			// // 第一个p元素
			// var p1 = pList.get(0);
			// console.log(p1);
			// var p3 = pList.get(2);
			// console.log(p3);


			console.log(pList);
			console.log('总共有几个：', pList.length);
			// for 循环遍历
			for (var i=0; i<pList.length; i++) {
				var item = pList[i];
				console.log(item)
			}
			// .each函数循环遍历
			console.log('---------------------')
			pList.each(function(index, value) {
				console.log(index, value)
			});

			console.log('--------------');  // json对象数组[{username: },{},]
			$.each(['a', 'bbb', 'ccc'], function(index, value) {
				console.log(index, value)
			})

			// .find的使用
			var list = pList.find('span');
			console.log(list)

			// 构建dom对象
			var htmlDom = $('<p class="test"/>');
			console.log(htmlDom)
			// 添加到html dom
			// htmlDom.appendTo('body');

			// $('body').append(htmlDom);


			// 在dom中添加内容
			// htmlDom.html('<span>我是新加的</span>');
			// $('body').append(htmlDom);

			// $('#beij').html('<span>我是新加的</span>');
			// .text() .val()

			// $('#beij').attr('class', '666');
			// 添加新的class
			$('#beij').addClass('666');
			// 移除class
			$('#beij').removeClass('area');

			// 操作css样式
			$('#beij').css({
				'color': '#0f0',
				'background-color': '#000'
			})

			// 隐藏元素
			// $('#info').hide();
			// 显示元素
			$('#info').show();


			// jQuery的链式调用
			var myDom = $('<p/>').text('你好').append('<span>，财主</span>').appendTo('body');
			// console.log(myDom)
		})
	</script>
</body>
</html>
```


