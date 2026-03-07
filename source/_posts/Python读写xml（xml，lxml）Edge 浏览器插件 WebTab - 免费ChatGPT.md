---
title: Python读写xml（xml，lxml）Edge 浏览器插件 WebTab - 免费ChatGPT
date: 2026-01-02 08:14:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Python读写xml（xml，lxml）Edge 浏览器插件 WebTab - 免费ChatGPT)
# XML
## 一、xml文件创建
### 方法一：使用xml.dom.minidom
#### 1、文件、标签的创建

```python
import xml.etree.ElementTree as etree
from xml.dom.minidom import Document
from xml.etree.ElementTree import Element as El

# 创建xml文件
doc = Document()
# 创建根节点
root_node = doc.createElement("root")
doc.appendChild(root_node)
# 创建子节点
son_node = doc.createElement("son_node")
root_node.appendChild(son_node)
# 子节点添加内容
text = doc.createTextNode("标签内容")
son_node.appendChild(text)
# 设置节点属性
son_node.setAttribute("name", "value")
son_node.setAttribute("name1", "value1")
# 添加二级子节点
sec_node = doc.createElement("second")
son_node.appendChild(sec_node)
text = doc.createTextNode("二级子节点内容")
sec_node.appendChild(text)
# 将内容保存到xml文件中
filename = "test.xml"
f = open(filename, "w", encoding="utf-8")
f.write(doc.toprettyxml(indent="  "))
f.close()
```
**输出**
```xml
<?xml version="1.0" ?>
<root>
  <son_node name="value" name1="value1">
    标签内容
    <second>二级子节点内容</second>
  </son_node>
</root>
```


### 方法二：使用ElementTree

```python
import xml.etree.ElementTree as etree

# 创建根元素
root = etree.Element("root")
# 创建子元素son,并设置子元素的标签名，属性
son = etree.SubElement(root, "max", attrib={"sex": "male"})
# 设置子元素son的内容
son.text = "content"
# 创建子元素的子元素sub_son
sub_son = etree.SubElement(son, "lily", attrib={"sex": "female"})
# 创建elementtree实例
et = etree.ElementTree(element=root)
et.write(r"test.xml", encoding="utf-8")
```
输出 
```xml
<root><max sex="male">content<lily sex="female" /></max></root>
```
## 二、xml文件修改
### 1、修改标签内容，属性
```python
# 修改标签的内容
tag.text = "modify_content"
# 修改标签的属性，或者添加属性
tag.set("atrri", "value3")
# 此操作将删除其他属性，只保留设置的属性
tag.attrib = {"atrri": "value"}
```
### 2、增加子标签
```python
tag = root.find(".//name")  # 需加入.//
# name标签下增加sex和address标签
sex = etree.SubElement(tag, "sex", attrib={"hobby": "swim"})
sex.text = "male"
addr = etree.SubElement(tag, "address", attrib={"provience": "guangdong"})
addr.text = "shenzhen"
tree.write(xml_path)
```
## 四、xml操作之删除
### 1、删除指定标签

```python
xml_path = r"test.xml"
tree = etree.parse(xml_path)  # 获取xml整个文档内容
for t in tree.iter():  # tree.iter（）可获得xml的所有节点及信息
    if t.tag == "entry":  # 查找到父节点
        print(list(t))
        for i in list(t):
            if i.tag == "category":  # 查找到子节点
                t.remove(i)  # 通过父节点删除子节点
                break  # 如果要删除父节点下所有子节点为category的，则为continue
tree.write(xml_path)
```
### 3、删除xml文件

```python
xml_path = r"test.xml"
os.remove(xml_path)
```
# LXML
lxml解析xml的时候，自动处理各种编码问题。而且它天生支持 XPath 1.0、XSLT 1.0、定制元素类。
## 1、读取xml文档
### 1）文档解析
lxml可以解析xml的字符串，使用etree.fromstring方法，如下所示：

```python
#coding:utf-8
from lxml import etree
 
xml_text = '<xml><head></head><body></body></xml>'
xml = etree.fromstring(xml_text)
```

lxml可以直接读取xml文件。

示例test.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<root version="1.2" tag="test">
    <head>
        <title>test xml document</title>
    </head>
    <body>
        <items id="1">
            <source>aa</source>
            <target>AA</target>
        </items>
        <items id="2">
            <source>bb</source>
            <target>BB</target>
        </items>
        <items id="3">
            <source>cc</source>
            <target id="3t">CC<bpt id="3t1"/>cc</target>
        </items>
    </body>
</root>
```

lxml读取xml文件的代码如下所示：

```python
#coding:utf-8
from lxml import etree
 
xml = etree.parse('test.xml') #读取test.xml文件
```
### 2）获取属性
根节点root中有两个属性，我们可以通过如下方法获取根节点和其属性：

```python
#coding:utf-8
from lxml import etree
 
xml = etree.parse('test.xml') #读取test.xml文件
root = xml.getroot() #获取根节点
 
#获取属性
print(root.items()) #获取全部属性和属性值
print(root.keys())  #获取全部属性
print(root.get('version', '')) #获取具体某个属性
```

得到如下结果：

```html
[('version', '1.0'), ('tag', 'test')]
['version', 'tag']
1.2
```

### 3）获取节点
假如我们不知道root节点下有什么节点，可以通过循环遍历。

```python
for node in root.getchildren():
    print(node.tag) #输出节点的标签名
```

得到如下结果：

```html
head
body
```
### 4）获取文本
有些元素中有文本，这个可以通过text属性获取。

```python
#获取source元素中的文本
for node in root.xpath('//source'):
    print(node.text)
```

## 2、写入xml文档
### 1）创建文档（节点）
对于lxml来说，任意节点都可以保存成一个xml文档。

我们只需要给该节点加入属性、内容、子节点等等即可。

那么创建节点方法如下：

```python
#coding:utf-8
from lxml import etree
 
#创建标签为root的节点
root = etree.Element('root')
```


在创建节点的同时，也可以给该节点加入命名空间：

```python
root = etree.Element('root', nsmap={'xmlns':'http://www.w3.org/1999/xhtml'})
```


在上面的test.xml中，还有两组属性。可用set方法添加属性：

```python
root.set('version', '1.2')
root.set('tag', 'test')
```

当然，也可以在创建节点的时候，就写入属性：

```python
attribs = {'version':'1.2', 'tag':'test'}
root = etree.Element('root', attrib=attribs)
```

### 2）添加子节点
添加根节点之后，根节点下有两个子节点：head和body。

添加子节点有两种方法，先看方法1：

```python
head = etree.Element('head')
root.append(head)
```

该方法是创建节点，再用append方法追加到root节点中。



还有一种方法，直接创建子节点：

```python
head = etree.SubElement(root, 'head')
```

推荐使用第2种方法，比较快捷。

若需要写属性值，除了用set方法。etree.SubElement方法也可以像etree.Element方法一样直接写入属性。

```python
head = etree.SubElement(root, 'head', attrib={'id':'head_id'})
```


### 3）添加文本
test.xml文档中，有几个地方需要添加文本。先给head添加title属性，并加入文本：

```python
title = etree.SubElement(head, 'title')
title.text = 'test xml document'
```

直接给text赋值即可。

### 4）保存文档
文档写好之后，就保存文档。保存文档这里有两种方法。

一种为通过etree.tostring方法得到xml的文本，再手动写入。这个方法过于麻烦，就不讲了，也不推荐。

常规方法是通过etree的tree对象保存文件。代码如下：

```python
#节点转为tree对象
tree = etree.ElementTree(root)
tree.write('test.xml', pretty_print=True, xml_declaration=True, encoding='utf-8', with_comments=True)
```

各个参数含义如下：

 1. 第1个参数是xml的完整路径(包括文件名)；
 2. pretty_print参数是否美化代码；
 3. xml_declaration参数是否写入xml声明，就是我们看到xml文档第1行文字；
 4. encoding参数很明显是保存的编码；
 5. with_comments参数是否保留注释。

**当你使用Python处理XML文件并且对注释进行修改时，你需要使用一个支持XML注释的XML解析器库。例如，使用Python内置的xml.etree.ElementTree来解析XML文档时，它是不会保留注释的，并且属性的顺序可能也会发生改变。但是，你可以使用第三方库lxml来处理XML文件并保留注释。**

```python
from lxml import etree

# 解析XML文件
doc = etree.parse('example.xml')

# 获取注释节点并进行修改
comment_node = doc.xpath('//comment()')[0]
comment_node.text = 'New comment'

# 保存修改后的XML文件，保留注释
doc.write('example.xml', encoding='utf-8', xml_declaration=True, pretty_print=True, with_comments=True)
```

**在代码中，我们使用etree.parse方法解析XML文件，然后通过doc.xpath方法获取注释节点并进行修改。最后使用doc.write方法保存修改后的XML文件，确保传递参数with_comments=True以保留注释。**

## 3、读取xml文件变成字符串和通过字符串生成xml文件

```python
from lxml import etree

def read_xml_file(file_path):
	with open(file_path, 'r', encoding='utf-8') as file:
		xml_string = file.read()
		
		return xml_string
		
template = read_xml_file('info.xml')
root = etree.fromstring(template)
with open('./qzk.xml', 'wb') as file:
	file.write(etree.tostring(root, pretty_print=True))
```

# Edge 浏览器插件 WebTab - 免费ChatGPT
![在这里插入图片描述](/img/bf1e962d9af92e012862d9dedbda718c.png)

# 视频逐帧保存图片

```python
import cv2
import os
# 打开视频文件
# 需要安装
# pip install opencv-python
path = r"***.mp4"
path_dir=r"pictures"
def get_frames():
    cap = cv2.VideoCapture(path)
    # 检查视频是否成功打开
    if not cap.isOpened():
        print("Error opening video stream or file")

    # 初始化帧计数器
    frame_count = 0

    # 循环逐帧读取视频
    while cap.isOpened():
        # 读取单帧
        ret, frame = cap.read()

        # 如果帧读取成功
        if ret:
            # 在这里添加对每一帧的处理代码

            # 保存当前帧为图像文件
            cv2.imwrite(os.path.join(path_dir,f"frame_{frame_count}.jpg"), frame)

            # 增加帧计数器
            frame_count += 1

        else:
            break

    # 清理资源
    cap.release()
    cv2.destroyAllWindows()
if __name__ == "__main__":
    get_frames()
```

