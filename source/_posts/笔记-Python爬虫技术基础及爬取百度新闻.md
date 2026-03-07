---
title: 笔记-Python爬虫技术基础及爬取百度新闻
date: 2026-01-05 08:12:00
tags:
  - "爬虫"
  - "Python"
  - "Requests"
  - "百度新闻"
  - "笔记"
categories:
  - "网络爬虫"
---

﻿
@[TOC](笔记-Python爬虫技术基础及爬取百度新闻)
​
## 1.1查看网页源代码
F12弹出来的东西叫做开发者工具，是进行数据挖掘的利器，对于爬虫来说，只需要会用下图的这两个按钮即可。
**（1）选择按钮**     
![在这里插入图片描述](/img/d7718e7cf9a244b4aaada9e8eb22fc81.png)
点击一下它，发现它会变成蓝色，然后把鼠标在页面上移动移动，会发现页面上的颜色机会发生改变。当移动鼠标的时候，会发现界面上的颜色会发生变化，并且Elements里的内容就会随之发生变化。
**(2) Elements元素按钮：**
![在这里插入图片描述](/img/bd7699f8200449d7a023550a810138e5.png)
“Elements”选项卡里面的内容可以理解为就是网页的源代码，最后爬虫爬到的内容大致是这样。
另外一个获取网页源码的方式是在网页上右击选择“**查看网页源代码**”。
## 1.2网页结构初步了解
结构其实很简单，就是一个大框套着一个小框，一个小框再套着一个小小框，**一般文本内容都是在最后的小框里**。
前两行的
![在这里插入图片描述](/img/24ccc2b6b79240e899fec9bcd9a6c053.png)

`<!DOCTYPE html>`与`<html>`是固定写法，作用是将代码声明为HTML文档。
`<body>`框表示主体信息，是最终展示在网页上的内容。<>包围起来的内容就是标签，例如，`<body>`读作body标签。通常写完`<body>`之后，最后得写一个`</body>`,表示一个框的闭合。
如果网页出现乱码（乱码就是中文显示成奇怪的符号），可以把**charset="utf-8"**中的**utf-8**改成**gbk**，这是两种不同的中文格式，各个浏览器可能各有不同。
## 1.3HTML基础知识 
### 1.标题`<h>`标签：
标题是通过`<h1> - <h6>` 标签来定义的，一般格式为：`<h1>`**标题内容**`</h1>`。其中h1的字号最大，h6的字号最小。
### 2.段落`<p>`标签：
段落是通过标签 `<p>` 来定义的,一般格式为：`<p>`**段落内容**`</p>`。
### 3.链接`<a>`标签：(定义链接)
链接是通过标签 `<a>` 来定义的，一般格式为：`<a href="链接地址">`文本内容`</a>`。如果想在一个新的标签页里打开百度首页，而不是把原网页覆盖了话，只要在”**链接地址**”的后面加上`target=_blank`即可
还有些常用的标签：定义表格的label标签、定义序号的`<li>`标签、定义图片的`<img>`标签、定义样式的`<script>`标签等
### 4.区块：
![在这里插入图片描述](/img/d97de5f6fec94345928abe9b40d01e0e.png)

区块最主要的表现形式就是`<div>**</div>`格式了，例：可以看到每个新闻都被包围在一个叫做`<div class ="result" id="*">***</div>`的框里，更加学术的说法来讲，这个`<div>**</div>`其实起到了一个分区的作用，**将百度新闻上这10条新闻分别放置了10个区域中**。
### 5.类（class）与 ID

 1. **类 (class)**: class的写法就是写在框的类型后面，比如`<h3 class="c-title">`以及`<div  class="result" id="1">`.
 2. ID：id的区分作用则更加每个class（类）可能相同，但是他们的id一般都不会相同
## 1.4百度新闻源代码获取
### 1.4.1获取网页源代码
![在这里插入图片描述](/img/275d77accaab4d0ea443c0bbe3477712.png)

通过requests库来尝试获取下百度新闻的网页源代码，代码如下：
![在这里插入图片描述](/img/3ac45974da324a1a9e8f1158f8867240.png)

获取到的源代码如下图所示：
可以看到其并没有获取到真正的网页源代码，这是因为这里的百度资讯网站
只认可浏览器发送过去的访问，而不认可直接通过Python发送过去的访问请求。
![在这里插入图片描述](/img/48e36f12d67c4ce5bb7d4876386d541c.png)

这时就需要设置下requests.get()中的headers参数，用来模拟浏览器进行访问。
![在这里插入图片描述](/img/b4a435ce76e14a7eb827b1b317991896.png)

运行结果如下图所示，可以发现此时已经获取到网页的源代码了：
这里的headers是一个字典，它的第一个元素的键名为’User-Agent'，值为'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36'。User-Agent实际上代表访问网站的浏览器是哪种浏览器。以谷歌浏览器为例讲解如何获取浏览器的User-Agent。打开谷歌浏节器,在地址栏中输入“about:version”，注意要用英文格式的冒号，按Enter键后在打开的界而中找到“用户代理”项，后面的字符串就是User-Agent.
对于之后的实战，只要记得在代码的最前面写上如下代码：
headers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36'}
然后每次用requests.get()访问网站时，加上headers=headers即可。
res = requests.get(url, headers=headers).text
### 1.4.2分析网页源代码信息
**三法之1: （1）F12方法：使用 选择按钮 和Elements**
![在这里插入图片描述](/img/c0ac42daa9f3459980e8942319b17f5a.png)

**三法之1: （2）使用2.1.2介绍的右击选择“查看网页源代码”**
然后便可以通过**Ctrl + F**快捷键（快速搜索快捷键）定位关心的内容了。
**三法之1: （3）在Python获得的网页源代码中查看**
然后便可以通过**Ctrl + F**快捷键（快速搜索快捷键）定位关心的内容了。
## 1.5正则表达式
### 1.5.1findall()函数
**正则表达式库re**
Findall()函数的功能是在原始文本中寻找所有符合匹配规则的文本内容，其使用格式如下所示：**re.findall(匹配规则，原始文本)**，匹配规则是由一个特定符号组成的字符串。findall()函数得到的是一个列表。
![在这里插入图片描述](/img/1574ad59ca6f47b5aabd4c4d4731bf34.png)

‘\d’表示匹配一个数字，‘\d\d\d’就表示匹配三个数字
### 1.5.2非贪婪匹配之(.*?)
![在这里插入图片描述](/img/aaa23aef87d440e4bbc78b731ac26c72.png)

简单来说**(.*?)**的作用就是来找到想要的内容，同时**不确定它的长度以及格式**，但是知道它在哪两块内容中间。其使用格式如下所示：**文本A(.*?)文本B**
### 1.5.3非贪婪匹配之 .*?
如果说**（.*?）**是用来**获取**文本A与文本B之间的内容；.*?的作用简单来说是**表示**文本C和文本D之间的内容。之所以要使用.*?，是因为文本C和文本D之间的内容经常变动或没有规律，无法写到匹配规则里；或者文本C和文本D之间的内容较多，我们不想写到匹配规则里。
### 1.5.4自动考虑换行的修饰符re.S
修饰符有很多，用的最多的就是**re.S**，它的作用就是在findall查找的时候，可以自动考虑到换行的影响，使得.*?可以匹配换行，使用格式如下：
**re.findall(匹配规则，原始文本，re.S)**
![在这里插入图片描述](/img/e802c8270b36449b9188fbd280cd8937.png)

获取的标题里包含了\n换行符，可以利用strip()函数清除换行符，代码如下：
![在这里插入图片描述](/img/dcaee56d38ac4abba61819ac85a35ff2.png)

注：如果想把title里的<em></em>给去掉，我们可以是原来学过的.replace 

 1. sub()函数
![在这里插入图片描述](/img/e558d284e66948ff919590072a852537.png)

sub()函数中的sub是英文substitute（替换）的缩写，其格式为：re.sub(需要替换的内容，替换值，原字符串)，该函数主要用于清洗正则表达式获取到的内容。
**这个方法不自会去掉`<em></em>`，还会去掉其它的<>里的内容。**

 1. 中括号[ ]的用法

中括号最主要的功能是使中括号里的内容不再有特殊含义。在正则表达式里，“.”“*”“?”等符号都有特殊的含义，但是如果想定位的就是这些符号，就需要使用中括号。

## 实例-百度新闻爬虫

```python
import requests
import re
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36'}

def baidu(keyword, page):  # 定义函数，方便之后批量调用
    num = (page - 1) * 10
    url = 'https://www.baidu.com/s?tn=news&rtt=4&bsst=1&cl=2&wd=' + keyword + '&pn=' + str(num)
    res = requests.get(url, headers=headers).text  # 通过requests库爬虫
 
    # 正则提取信息
    p_href = '<h3 class="c-title">.*?<a href="(.*?)"'
    p_title = '<h3 class="c-title">.*?>(.*?)</a>'
    p_info = '<p class="c-author">(.*?)</p>'
    href = re.findall(p_href, res, re.S)
    title = re.findall(p_title, res, re.S)
    info = re.findall(p_info, res, re.S)

    # 数据清洗
    source = []
    date = []
    for i in range(len(title)):
        title[i] = title[i].strip()
        title[i] = re.sub('<.*?>', '', title[i])
        info[i] = re.sub('<.*?>', '', info[i])
        source.append(info[i].split('&nbsp;&nbsp;')[0])  
        date.append(info[i].split('&nbsp;&nbsp;')[1])
        source[i] = source[i].strip()
        date[i] = date[i].strip()
 
    # 通过字典生成二维DataFrame表格   
    result = pd.DataFrame({'关键词': keyword, '标题': title, '网址': href, '来源': source, '日期': date})
    return result
 
# 通过pandas库将数据进行整合并导出为Excel
import pandas as pd  
df = pd.DataFrame()
 
keywords = ['华能信托', '人工智能', '科技', '体育', 'Python', '娱乐', '文化', '阿里巴巴', '腾讯', '京东']
for keyword in keywords:
    for i in range(10):  # 循环10遍，获取10页的信息
        result = baidu(keyword, i+1)
        df = df.append(result)  # 通过append()函数添加每条信息到df中
        print(keyword + '第' + str(i+1) + '页爬取成功')

df.to_excel('新闻_new.xlsx')  # 在代码所在文件夹生成EXCEL文件
```

