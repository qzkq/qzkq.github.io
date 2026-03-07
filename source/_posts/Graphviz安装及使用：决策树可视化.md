---
title: Graphviz安装及使用：决策树可视化
date: 2026-01-01 08:09:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿## Python大数据分析与机器学习学习笔记
Graphviz：[微信公众号（数学建模与人工智能）](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&amp;mid=100000371&amp;idx=1&amp;sn=00b88a37f1d7b7f06b32607f4ed0adc8&amp;scene=19&token=1933614188&lang=zh_CN#wechat_redirect)
github地址：[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
### 1、决策树模型搭建

```python
# 模型搭建代码汇总
import pandas as pd
# 1.读取数据与简单预处理
df = pd.read_excel('员工离职预测模型.xlsx')
df = df.replace({'工资': {'低': 0, '中': 1, '高': 2}})
# 2.提取特征变量和目标变量
X = df.drop(columns='离职')
y = df['离职']
# 3.划分训练集和测试集
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=123)
# 4.模型训练及搭建
from sklearn.tree import DecisionTreeClassifier
model = DecisionTreeClassifier(max_depth=3, random_state=123)
model.fit(X_train, y_train)
```

### 2、graphviz插件安装与环境变量部署
本小节主要讲解一下graphviz插件的安装与环境变量部署，为之后将决策树模型可视化做准备。
2.1 graphviz插件下载
搭建完决策树模型后，我们可以通过graphviz插件将其可视化呈现出来。首先需要安装一下graphviz插件，其下载地址为：https://graphviz.gitlab.io/download/，以Windows版本为例，在下载网站上选择下图框中内容：Stable 2.38 Windows install packages。
 ![在这里插入图片描述](/img/8acd432b5aec9825dffdc438b164b8df.png)

然后下载下图所示的msi文件：
 ![在这里插入图片描述](/img/67c3775b165266e19433d819226142b7.png)

下载完该msi文件后进行安装，注意，要记住下图所示的安装的文件路径，之后进行环境变量部署的时候会用到。
 ![在这里插入图片描述](/img/32ba962d7c6b7ddbe27b2ebaf89f0612.png)
2.2 环境变量部署
安装完graphviz后我们需要进行环境变量部署，所谓环境变量部署，就是把安装的软件部署到整个电脑系统环境中，这样在电脑的各个地方都可以调用配置好的软件。其配置方法如下：
1.右键点击我的电脑，选择“属性”选项，如下图所示：
 ![在这里插入图片描述](/img/3dc9dff53cb9ea016f29ae376ae49710.png)

2.在弹出的界面中选择“高级系统设置”，如下图所示：
 ![在这里插入图片描述](/img/fdccd10c98d466dc31cbfbf9c01272c6.png)

3.如下图所示，在弹出的系统属性界面中，选择“环境变量”，然后在弹出的“环境变量”弹出窗内选择“系统变量”中的Path那一行，然后点击“编辑”按钮。
 ![在这里插入图片描述](/img/b4213392af1dab3937a5717cd20e168f.png)

4.如下图所示，在弹出的界面中，点击“新建”，然后将graphviz安装所在文件夹中bin文件夹添加到其中，最后点击确定即可。环境变量部署就是将软件所在文件路径中的bin文件夹，如这里的“C:\Program Files (x86)\Graphviz2.38\bin”添加到上图所示的Path中，所以当完成这一步后，环境变量也就部署完毕了。
 ![在这里插入图片描述](/img/c38bbbc98a1d1801f911b3273b9b0379.png)

### 3、在Python中使用graphviz
上面已经安装好graphviz插件，并完成环境变量部署后，我们就可以在Python中使用graphviz将决策树模型可视化输出出来了。
3.1 graphviz库安装
想要在Python中使用graphviz插件，首先还需要在Python中安装相关的库，这里同样是通过PIP安装法进行安装，以Windows系统为例，Win + R组合键调出系统运行框，输入cmd后点击确定，在弹出框内输入pip install graphviz，按一下回车键等待安装结束即可。如下图所示：
 ![在这里插入图片描述](/img/f8c19f9baf2c5bf0b4d5387f379d56db.png)

3.2 graphviz库的使用
通过如下代码就可以生成一个可视化的决策树模型：

```python
from sklearn.tree import export_graphviz
import graphviz
dot_data = export_graphviz(model, out_file=None, class_names=['0', '1'])
graph = graphviz.Source(dot_data)
graph.render('决策树可视化')
```

前两行引入使用graphviz的相关库，第三行通过export_graphviz()方法将之前搭建的决策树模型model转换为字符串格式并赋值给dot_data，其中注意需要设定out_file参数为None，这样获得的才是字符串格式，感兴趣的读者可以将dot_data打印出来看下，获得的dot_data如下图所示，里面的内容其实就是之后要可视化的内容。
 ![在这里插入图片描述](/img/8ce9dc252c266127275ceedccc4e88d3.png)

第四行代码则是将dot_data转换成可视化的格式，第五行代码则是通过render()方法将图像输出出来，通过上述代码默认是输出一个PDF文件，PDF文件如下图所示：
 ![在这里插入图片描述](/img/95d03fe8d2b2e6a8de75709aad2f2857.png)

可以看到里面的内容的确就是上面dot_data文本内容的可视化呈现了。其中X[1]就表示第2个特征变量：满意度，X[3]则表示第4个特征变量：工程数量，X[5]则表示第6个特征变量：工龄；gini则表示该节点的基尼系数；samples则表示该节点中的样本数，比如说第一个节点，也即根节点中的12000也即训练集中的样本数量；value则表示不同种类所占的个数，比如说根节点中value左边的9120则表示非流失客户的数量，2880则表示流失客户的数量，class=0则是认为该节点为未流失节点。
如果通过运行上述代码产生下图所示的报错信息，说明之前的环境变量部署没有部署成功。
 ![在这里插入图片描述](/img/0e5a2ac9f8c0771489ecd662c6fec6ef.png)

那么此时我们可以手动在程序里部署环境变量，只要在代码最上方添加如下两行代码即可。其中添加字母r的原因是用来去除文件路径中反斜杠的特殊含义，另一种方法是用两个反斜杠“\\”代替单个斜杠“\”。如果graphviz的安装路径不是下面所示，那么自行修改即可。

```python
import os 
os.environ['PATH'] = os.pathsep + r'C:\Program Files (x86)\Graphviz2.38\bin'
```

完整代码如下：

```python
from sklearn.tree import export_graphviz
import graphviz
import os # 以下这两行是手动进行环境变量配置，防止在本机的环境变量部署失败
os.environ['PATH'] = os.pathsep + r'C:\Program Files (x86)\Graphviz2.38\bin'
dot_data = export_graphviz(model, out_file=None)
graph = graphviz.Source(dot_data)
graph.render('决策树可视化')
```

如果想让可视化的图片里的内容更丰富些，可以在export_graphviz()的括号中增添一些参数，比如增加feature_names参数可以显示特征变量的名称，不过直接使用graphviz识别不了中文，所以我们等会先用英文来做演示；增添class_names参数则可以显示最后的分类结果，因为中文可能会出现乱码问题，所以我们用字符串'0'和'1'来代替'流失'和'非流失'，注意不能写成数字格式的0和1，因为class_names参数中只能设置字符串格式的数据；将filled参数设置为True还可以给可视化的决策树添加颜色，演示代码如下：

```python
# 添加名称（feature_names）和填充颜色（filled=True）
dot_data = export_graphviz(model, out_file=None, feature_names=['income', 'satisfication', 'score', 'project_num', 'hours', 'year'], class_names=['0', '1'], filled=True)
```

此时生成的可视化图形如下图所示：
 ![在这里插入图片描述](/img/f8eff50d194ca91aa442b11b4f1ddb9f.png)

此时的特征变量就不再试通过X[4]这种形式来表示了，而是通过具体我们设置的名称来显示了。又因为我们设置了class_names，所以每一个节点多出一个内容叫作class，其中0代表着非流失，1代表着流失，其判断依据为value中哪个类别占的数量多，则判定为该类别，比如根节点中的value，非流失客户为475人，流失客户为325人，那么则判断该节点的类别为0，及非流失。在实际应用中，我们更关心最终叶子节点的分类类别，比如左下角的叶子节点的分类类别为1，及流失，那么如果新来一个客户，通过该决策树模型分到左下角的叶子节点的话，那么则判断其为流失客户。
上面我们生成的可视化图形已经比较完善了，但是graphviz本身是没有办法识别中文的，所以如果把上面代码中的feature_names或者class_names参数中的内容写成中文的话，在最后生成图形中会出现中文乱码的问题。关于中文乱码的问题我们将在下一小节解决。
3.3 中文乱码问题解决办法
这一小节我们重点解决graphviz因为不能识别中文，导致生成的中文乱码问题。步骤如下：
1.引入相关库并在程序中部署好环境变量

```python
from sklearn.tree import export_graphviz
import os # 以下这两行是手动进行环境变量配置，防止在本机的环境变量部署失败
os.environ['PATH'] = os.pathsep + r'C:\Program Files (x86)\Graphviz2.38\bin'
```

这里不需要引入graphviz库，因为等会生成图形的方式不再是利用上一小节graphviz.Source(dot_data)来进行生成了，这个稍后再讲。此外，这里通过手动的方式进行了环境变量部署，也是为了防止之前在本机的环境变量部署失败。
2.生成dot_data

```python
dot_data = export_graphviz(model, out_file=None, feature_names=X_train.columns, class_names=['不流失', '流失'], rounded=True, filled=True)
```

这里通过export_graphviz()方法生成dot_data，此时的dot_data是一个字符串类型的数据，里面的内容则是之后要进行可视化的内容，这个在上一小节也提到过。这里注重讲下export_graphviz()方法里设置的参数：首先model表示之前训练的模型，我们的目的就是将其转换成字符串的格式然后再转换成图形；out_file参数是用来设置生成的内容为字符串的；feature_names则是用来设置特征变量的名称，其中X_train.columns则是测试训练集的表头名称，也即这个决策树模型的特征变量名称们，感兴趣的读者可以将其打印出来，效果如下，可以看到这里的表头全是中文，如果还用之前的方式进行可视化的话，其中的中文会出现乱码现象。
Index(['收入', '年龄', '性别', '历史授信额度', '历史流失次数'], dtype='object')
class_names则是设定最后的分类结果，这里我们直接设置成中文的'不流失'和'流失'，之后会讲如何处理中文乱码问题；rounded参数需要设置成True，这样之后设置中文格式才会生效；最后的filled参数设置为True可以使得生产的决策树有颜色。
 ![在这里插入图片描述](/img/3fcc8d26d6e6c266a83bf519c2a2a12e.png)

3.将dot_data写入到txt文件中
通过如下代码可以将之前的dot_data字符串类型的数据写到txt文件中。

```python
f = open('dot_data.txt', 'w')
f.write(dot_data)
f.close()
```

之后我们将利用这里生成的dot_data.txt文件来生成新的决策树文本文件并修改其中的中文配置信息从而实现通过graphviz能够输出中文的目的。此时生成dot_data.txt文件如下图所示，可以看到里面的内容的确就是之后可视化图形要展示的的内容了。
 ![在这里插入图片描述](/img/00c8df23155450381a7b0a2bf9f5b12f.png)

4.修改字体设置
通过如下代码，我们可以修改原来dot_data.txt中的字体及编码格式，并将其另存为一个新的dot_data_new.txt文件，为之后解决中文乱码问题做铺垫。

```python
import re
f_old = open('dot_data.txt', 'r')
f_new = open('dot_data_new.txt', 'w', encoding='utf-8')
for line in f_old:
    if 'fontname' in line:
        font_re = 'fontname=(.*?)]'
        old_font = re.findall(font_re, line)[0]
        line = line.replace(old_font, 'SimHei')
    f_new.write(line)
f_old.close()
f_new.close()
```

这里首先引入正则表达式库re，为之后的替换字体做准备；然后我们通过open()函数读取原来的dot_data.txt中的文本内容，其中'r'表示以读取模式打开txt文件，我们将文件内容赋值给fold变量；之后通过open()函数新建一个dot_data_new.txt文件，其中'w'表示以写入模型打开txt文件，并且每次写入时都会把原有内容清除，encoding参数为编码方式，这里设置为能够支持中文显示的utf-8编码，这个编码方式的设置对解决中文乱码问题而言很重要。
下面的几行代码这里单独拎出来讲解一下：

```python
for line in f_old:
if 'fontname' in line:
font_re = 'fontname=(.*?)]'
old_font = re.findall(font_re, line)[0]
line = line.replace(old_font, 'SimHei')
f_new.write(line)
```

这几行代码的目的就是用来替换原来dot_data.txt的默认的字体格式，替换成SimHei也即黑体格式，如下图所示：
 ![在这里插入图片描述](/img/fe855ce2ceecaabfde49a39e819d3576.png)

这里首先通过for循环语句遍历f_old，也即dot_data.txt中的每一行，然后通过if判断语句来查看每一行中是否有fontname内容，其中fontname的中文意思就是字体名字，对于含有fontname的行，我们通过正则表达式的非贪婪匹配(.*?)将默认的字体找到，并通过replace()函数将旧的字体替换成我们想设置的字体，比如这里设置的SimHei，也即黑体字体。最后将这些替换完的新的行写入到新的txt文件f_new中。
最后生成完新的f_new文件后，我们就可以通过close()函数将f_old和f_new关闭。
补充知识点：这里的SimHei是黑体的英文翻译，如果想采用其他字体，可参考下面的字体英文对照表：

 - 黑体	SimHei 
 - 微软雅黑	Microsoft YaHei 
 - 新宋体	NSimSun 
 - 新细明体	PMingLiU 
 - 细明体	MingLiU
 - 仿宋	FangSong 
 - 楷体	KaiTi

5.生成可视化文件
生成完新的重新设置编码格式及字体的dot_data_new.txt文件后，我们就可以通过graphviz插件生成可视化的文件了，代码如下：

```python
os.system('dot -Tpdf dot_data_new.txt -o 决策树模型.pdf')
```

这里是通过调用os.system()系统函数来使用graphviz插件，其中-Tpdf表示生成PDF格式的文件，如果将其改成-Tpng生成PNG格式的图片文件，代码如下：

```python
os.system('dot -Tpng dot_data_new.txt -o 决策树模型.png')
```

最后的生成效果如下图所示，可以看到已经可以显示中文了。
 ![在这里插入图片描述](/img/0fbfb8da22ef75cc9e1ec5036b900cad.png)

所有代码整理如下：

```python
from sklearn.tree import export_graphviz
import graphviz
import os # 以下这两行是手动进行环境变量配置，防止在本机环境的变量部署失败
os.environ['PATH'] = os.pathsep + r'C:\Program Files (x86)\Graphviz2.38\bin'
# 生成dot_data
dot_data = export_graphviz(model, out_file=None, feature_names=X_train.columns, class_names=['不流失', '流失'], rounded=True, filled=True) # rounded和字体有关，filled设置颜色填充
# 将生成的dot_data内容导入到txt文件中
f = open('dot_data.txt', 'w')
f.write(dot_data)
f.close()
# 修改字体设置，避免中文乱码！
import re
f_old = open('dot_data.txt', 'r')
f_new = open('dot_data_new.txt', 'w', encoding='utf-8')
for line in f_old:
    if 'fontname' in line:
        font_re = 'fontname=(.*?)]'
        old_font = re.findall(font_re, line)[0]
        line = line.replace(old_font, 'SimHei')
    f_new.write(line)
f_old.close()
f_new.close()
# 以PNG的图片形式存储生成的可视化文件
os.system('dot -Tpng dot_data_new.txt -o 决策树模型.png')  
print('决策树模型.png已经保存在代码所在文件夹！')
# 以PDF的形式存储生成的可视化文件
os.system('dot -Tpdf dot_data_new.txt -o 决策树模型.pdf')  
print('决策树模型.pdf已经保存在代码所在文件夹！')
```

