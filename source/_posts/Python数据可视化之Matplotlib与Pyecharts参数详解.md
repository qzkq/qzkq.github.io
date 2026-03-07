---
title: Python数据可视化之Matplotlib与Pyecharts参数详解
date: 2026-01-02 08:06:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿# Python数据可视化
[ 微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&amp;mid=100000296&amp;idx=1&amp;sn=e63c3bb1446eb113c2cd82660997f9db&amp;scene=19&token=1933614188&lang=zh_CN#wechat_redirect)
在Matplotlib中，设置线的颜色（color）、标记（marker）、线型（line）等参数。

```python
线的颜色	颜色
'b'	蓝色
'g'	绿色
'r'	红
'y'	黄色
'k'	黑
'w'	白色
```

```python
线的标记	描述
'.'	点标记
','	像素标记
'o'	圆圈标记
's'	方形标记
'p'	五角大楼标记
'*'	星形标记
'+'	加号标记
'x'	x 标记
'D'	钻石标记
```

```python
线的类型	描述
'-'	  实线样式
'--'  虚线样式
'-.'  破折号-点线样式
':'	  虚线样式
```

Matplotlib坐标轴的刻度设置，可以使用plt.xlim()和plt.ylim()函数，参数分别是坐标轴的最小最大值。
在Matplotlib中，可以使用plt.xlabel()函数对坐标轴的标签进行设置，其中参数xlabel设置标签的内容、size设置标签的大小、rotation设置标签的旋转度、horizontalalignment（水平对齐）设置标签的左右位置（分为center、right和left）、verticalalignment（垂直对齐）设置标签的上下位置（分为center、top和bottom）。
图例是集中于地图一角或一侧的地图上各种符号和颜色所代表内容与指标的说明，有助于更好的认识图形。在Matplotlib中，图例的设置可以使用plt.legend()函数，我们还可以重新定义图例的内容、位置、字体大小等参数。

```python
Matplotlib图例的主要参数配置如下：
plt.legend(loc,fontsize,frameon,ncol,title,shadow,markerfirst,markerscale,numpoints,fancybox,framealpha,borderpad,labelspacing,handlelength,bbox_to_anchor,*)
```

```python
属性	           说明
Loc	        图例位置，如果使用了bbox_to_anchor参数，则该项无效。
Fontsize	设置字体大小。
Frameon	    是否显示图例边框。
Ncol	    图例的列的数量，默认为1。
Title	    为图例添加标题。
Shadow	    是否为图例边框添加阴影。
Markerfirst	True表示图例标签在句柄右侧，False反之。
Markerscale	图例标记为原图标记中的多少倍大小。
Numpoints	表示图例中的句柄上的标记点的个数，一般设为1。
Fancybox	是否将图例框的边角设为圆形。
Framealpha	控制图例框的透明度。
Borderpad	图例框内边距。
Labelspacing	图例中条目之间的距离。
Handlelength	图例句柄的长度。
bbox_to_anchor	如果要自定义图例位置需要设置该参数。
```

## 标题基础图表函数

```python
函数	       说明
plt.plot()	绘制坐标图
plt.boxplot()	绘制箱形图
plt.bar()	绘制条形图
plt.barh()	绘制横向条形图
plt.polar()	绘制极坐标图
plt.pie()	绘制饼图
plt.psd()	绘制功率谱密度图
plt.specgram()	绘制谱图
plt.cohere()	绘制相关性函数
plt.scatter()	绘制散点图
plt.hist()	绘制直方图
plt.stem()	绘制柴火图
plt.plot_date()	绘制数据日期
```

```python
Matplotlib绘制直方图，使用plt.hist()这个函数，函数参数如下：
Matplotlib.pyplot.hist(x,bins=None,range=None,density=None,weights=None, cumulative=False, bottom=None, histtype='bar', align='mid', orientation='vertical', rwidth=None, log=False, color=None, label=None, stacked=False, normed=None, *, data=None, **kwargs)
```

```python
属性	说明
X	    指定要绘制直方图的数据。
Bins	指定直方图条形的个数。
Range	指定直方图数据的上下界，默认包含绘图数据的最大值和最小值。
Density	若为True，返回元组的第一个元素将是归一化的计数，以形成概率密度。
Weights	该参数可以为每一个数据点设置权重。
Cumulative	是否需要计算累计频数或频率。
Bottom	可以为直方图的每个条形添加基准线，默认为0。
Histtype	指定直方图的类型，默认为bar，还有’barstacked’、‘step’等。
Align	设置条形边界值的对其方式，默认为mid，还有’left’和’right’。
Orientation	设置直方图的摆放方向，默认为垂直方向。
Rwidth	设置直方图条形宽度的百分比。
Log	    是否需要对绘图数据进行对数变换。
Color	设置直方图的填充色。
Label	设置直方图的标签，可以通过legend展示其图例。
Stacked	当有多个数据时，是否需要将直方图呈堆叠摆放，默认水平摆放。
Normed	已经弃用，改用density参数。
```

```python
	Matplotlib绘制折线图，使用plt.plot()这个函数，函数参数如下：
	plot([x], y, [fmt], data=None, **kwargs)
```

```python
属性	说明
x,y	    设置数据点的水平或垂直坐标。
Fmt	    用一个字符串来定义图的基本属性如颜色，点型，线型。
Data	带有标签的绘图数据。
```

```python
	Matplotlib绘制条形图，使用plt.bar()这个函数，函数参数如下：
	Matplotlib.pyplot.bar(x,height,width=0.8,bottom=None,*,align='center',data=None, **kwargs)
```

```python
属性	说明
X	    设置横坐标。
Height	条形的高度。
Width	直方图宽度，默认0.8。
Botton	条形的起始位置。
Align	条形的中心位置。
Color	条形的颜色。
Edgecolor	边框的颜色。
Linewidth	边框的宽度。
tick_label	下标的标签。
Log	    y轴使用科学计算法表示。
Orientation	是竖直条还是水平条。
```

```python
	Matplotlib绘制饼图，使用plt.pie()这个函数，函数参数如下：
	Matplotlib.pyplot.pie(x, explode=None, labels=None, colors=None, autopct=None, pctdistance=0.6, shadow=False, labeldistance=1.1, startangle=None, radius=None, counterclock=True, wedgeprops=None, textprops=None, center=(0, 0), frame=False, rotatelabels=False, *, data=None)
```

```python
属性	说明
X	    每一块的比例，如果sum(x) > 1则会进行归一化处理。
Labels	每一块饼图外侧显示的说明文字。
Explode	每一块离开中心的距离。
Startangle	起始绘制角度，默认图是从x轴正方向逆时针画起，如设定=90则从y轴正方向画起。
Shadow	在饼图下面画一个阴影。默认为False，即不画阴影。
Labeldistance	label标记的绘制位置，相对于半径的比例，默认值为1.1， 如<1则绘制在饼图内侧。
Autopct	控制饼图内百分比设置。
Pctdistance	类似于labeldistance，指定autopct的位置刻度，默认值为0.6。
Radius	控制饼图半径，默认值为1。
Counterclock	指定指针方向，可选，默认为True，即逆时针。
Wedgeprops	字典类型，可选，默认值None。参数字典传递给wedge对象用来画饼图。
Textprops	设置标签和比例文字的格式，字典类型，可选，默认值为None。
Center	浮点类型的列表，可选，默认值(0，0)。图标中心位置。
Frame	布尔类型，可选，默认为False。如果是True，绘制带有表的轴框架。
Rotatelabels	布尔类型，可选，默认为False。如果为True，旋转每个label到指定的角度。
```

```python
	Matplotlib绘制散点图用到plt.scatter()这个函数，函数参数如下：
	Matplotlib.pyplot.scatter(x, y, s=None, c=None, marker=None, cmap=None, norm=None, vmin=None, vmax=None, alpha=None, linewidths=None, verts=None, edgecolors=None, *, data=None, **kwargs)
```

```python
属性	说明
x,y	绘图的数据，都是向量且必须长度相等。
S	设置标记大小。
C	设置标记颜色。
marker	设置标记样式。
cmap	设置色彩盘。
norm	设置亮度，为0到1之间。
vmin，vmax	设置亮度，如果norm已设置，该参数无效。
alpha	设置透明度，为0到1之间。
linewidths	设置线条的宽度。
edgecolors	设置轮廓颜色。
```

```python
	Matplotlib绘制箱线图用plt.boxplot()这个函数，函数参数如下：
	plt.boxplot(x,notch=None,sym=None,vert=None,whis=None,positions=None,widths=None,patch_artist=None,meanline=None,showmeans=None,showcaps=None,showbox=None,showfliers=None,boxprops=None,labels=None,flierprops=Non，medianprops=None,meanprops=None, capprops=None,whiskerprops=None)
```

```python
属性	说明
X	指定要绘制箱线图的数据。
notch	是否是凹口的形式展现箱线图，默认非凹口。
sym	指定异常点的形状，默认为+号显示。
vert	是否需要将箱线图垂直摆放，默认垂直摆放。
whis	指定上下须与上下四分位的距离，默认为1.5倍的四分位差。
positions	指定箱线图的位置，默认为[0，1，2…]。
widths	指定箱线图的宽度，默认为0.5。
patch_artist	是否填充箱体的颜色。
meanline	是否用线的形式表示均值，默认用点来表示。
showmeans	是否显示均值，默认不显示。
showcaps	是否显示箱线图顶端和末端的两条线，默认显示。
showbox	是否显示箱线图的箱体，默认显示。
showfliers	是否显示异常值，默认显示。
boxprops	设置箱体的属性，如边框色，填充色等。
labels	为箱线图添加标签，类似于图例的作用。
filerprops	设置异常值的属性，如异常点的形状、大小、填充色等。
medianprops	设置中位数的属性，如线的类型、粗细等。
meanprops	设置均值的属性，如点的大小、颜色等。
capprops	设置箱线图顶端和末端线条的属性，如颜色、粗细等。
whiskerprops	设置须的属性，如颜色、粗细、线的类型等。
```

Pyecharts可以方便的绘制一些基础视图，包括折线图、条形图、箱形图、涟漪散点图、K线图以及双坐标轴图等
	折线图是用直线段将各个数据点连接起来而组成的图形，以折线方式显示数据的变化趋势。折线图可以显示随时间（根据常用比例设置）而变化的连续数据，因此非常适合显示相等时间间隔的数据趋势。在折线图中，类别数据沿水平轴均匀分布，值数据沿垂直轴均匀分布。例如为了显示不同订单日期的销售额走势，可以创建不同订单日期的销售额折线图。

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
y_axis	系列数据。
is_selected	是否选中图例。
is_connect_nones	是否连接空数据，空数据使用 `None` 填充。
xaxis_index	使用的x轴的index，在单个图表实例中存在多个x轴的时候有用。
yaxis_index	使用的y轴的index，在单个图表实例中存在多个 y 轴的时候有用。
color	系列 label 颜色。
is_symbol_show	是否显示 symbol, 如果 false 则只有在 tooltip hover 的时候显示。
symbol	标记的图形。
symbol_size	标记的大小，可以设置成诸如 10 这样单一的数字，也可以用数组分开表示宽和高。
stack	数据堆叠，同个类目轴上系列配置相同的　stack　值可以堆叠放置。
is_smooth	是否平滑曲线。
is_step	是否显示成阶梯图。
markpoint_opts	标记点配置项。
markline_opts	标记线配置项。
tooltip_opts	提示框组件配置项。
label_opts	标签配置项。
linestyle_opts	线样式配置项。
areastyle_opts	填充区域配置项。
itemstyle_opts	图元样式配置项。
```

	条形图是一种把连续数据画成数据条的表现形式，通过比较不同组的条形长度，从而对比不同组的数据量大小，描绘条形图的要素有3个：组数、组宽度、组限。绘画条形图时，不同组之间是有空隙的。条形用来比较两个或以上的价值（不同时间或者不同条件），只有一个变量，通常利用于较小的数据集分析。条形图亦可横向排列，或用多维方式表达。

```python
属性	说明
series_name	系列名称，用于tooltip的显示，legend的图例筛选。
yaxis_data	系列数据。
is_selected	是否选中图例。
xaxis_index	使用的x轴的index，在单个图表实例中存在多个x轴的时候有用。
yaxis_index	使用的y轴的index，在单个图表实例中存在多个y轴的时候有用。
color	系列label颜色。
stack	数据堆叠，同个类目轴上系列配置相同的stack值可以堆叠放置。
category_gap	同一系列的柱间距离，默认为间距的20%，表示柱子宽度的20%。
gap	如果想要两个系列的柱子重叠，可以设置gap为'-100%'。
label_opts	标签配置项。
markpoint_opts	标记点配置项。
markline_opts	标记线配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

条形图的数据项在BarItem类中进行设置

```python
属性	说明
name	数据项名称。
value	单个数据项的数值。
label_opts	单个柱条文本的样式设置。
itemstyle_opts	图元样式配置项。
tooltip_opts	提示框组件配置项。
```

	箱形图又称箱线图，是一种用作显示一组数据分散情况资料的统计图。因形状如箱子而得名。在各种领域也经常被使用，常见于品质管理。
	箱形图主要用于反映原始数据分布的特征，还可以进行多组数据分布特征的比较。箱线图的绘制方法是：先找出一组数据的上边缘、下边缘、中位数和两个四分位数；然后， 连接两个四分位数画出箱体；再将上边缘和下边缘与箱体相连接，中位数在箱体中间。

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
y_axis	系列数据。
is_selected	是否选中图例。
xaxis_index	使用的 x 轴的 index，在单个图表实例中存在多个 x 轴的时候有用。
yaxis_index	使用的 y 轴的 index，在单个图表实例中存在多个 y 轴的时候有用。
label_opts	标签配置项。
markpoint_opts	标记点配置项。
markline_opts	标记线配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

	涟漪散点图是一类特殊的散点图，只是散点图中带有涟漪特效，利用特效可以突出显示某些想要的数据。

```python
属性	说明
series_name	系列名称，用于tooltip的显示，legend的图例筛选。
y_axis	系列数据。
is_selected	是否选中图例。
xaxis_index	使用的x轴的index，在单个图表实例中存在多个x轴的时候有用。
yaxis_index	使用的y轴的index，在单个图表实例中存在多个y轴的时候有用。
color	系列label颜色。
symbol	标记的图形。
symbol_size	标记的大小，可以设置成诸如10这样单一的数字，也可以用数组分开表示宽和高。
label_opts	标签配置项。
markpoint_opts	标记点配置项。
markline_opts	标记线配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

```python
属性	说明
series_name	 系列名称，用于 tooltip 的显示，legend 的图例筛选。
y_axis	系列数据。
is_selected	是否选中图例。
xaxis_index	使用的 x 轴的 index，在单个图表实例中存在多个 x 轴的时候有用。
yaxis_index	使用的 y 轴的 index，在单个图表实例中存在多个 y 轴的时候有用。
color	系列 label 颜色。
symbol	标记图形形状。
symbol_size	标记的大小。
label_opts	标签配置项。
effect_opts	涟漪特效配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

	K线图又称蜡烛图，股市及期货市场中的K线图的画法包含四个数据，即开盘价、最高价、最低价、收盘价，所有的k线都是围绕这四个指标展开，反映股票的状况。如果把每日的K线图放在一张纸上，就能得到日K线图，同样也可画出周K线图、月K线图。

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
y_axis	系列数据。
is_selected	是否选中图例。
xaxis_index	使用的 x 轴的 index，在单个图表实例中存在多个 x 轴的时候有用。
yaxis_index	使用的 y 轴的 index，在单个图表实例中存在多个 y 轴的时候有用。
markline_opts	标记线配置项。
markpoint_opts	标记点配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

	双坐标轴图是一种组合图表，一般将两种不同类型图表组合在同一个“画布”上，如柱状图和折线图的组合；当然也可将类型相同而数据单位不同的图表组合在一起。双坐标轴图中最难画的应该是“柱状图”与“柱状图”的组合，因为会遇到同一刻度对应“柱子”与“柱子”完全互相重叠的问题。
Pyecharts可以生成一些比较复杂的视图，包括日历图、漏斗图、仪表盘、环形图、雷达图、旭日图等。
	日历图是一个日历数据视图，提供一段时间的日历布局，使我们可以更好地查看所选日期每一天的数据。
	日历图的参数配置。
	日历图坐标系组件的配置项。

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
yaxis_data	系列数据，格式为 [(date1, value1), (date2, value2), ...]。
is_selected	是否选中图例。
label_opts	标签配置项。
calendar_opts	日历坐标系组件配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

```python
属性	说明
pos_left	calendar组件离容器左侧的距离。
pos_top	calendar组件离容器上侧的距离。
pos_right	calendar组件离容器右侧的距离。
pos_bottom	calendar组件离容器下侧的距离。
orient	日历坐标的布局朝向。可选：'horizontal','vertical'。
range_	必填，日历坐标的范围。
daylabel_opts	星期轴的样式。
monthlabel_opts	月份轴的样式。
yearlabel_opts	年份的样式。
```

	漏斗图又叫倒三角图，适用于业务流程比较规范、周期长、环节多的流程分析,通过漏斗各环节业务数据的比较,能够直观地发现和说明问题所在，还可以应用于对数据从某个维度上进行比较。

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
data_pair	系列数据项，格式为 [(key1, value1), (key2, value2)]。
is_selected	是否选中图例。
color	系列 label 颜色。
sort_	数据排序，可以取 'ascending'，'descending'，'none'（表示按 data 顺序）。
gap	    数据图形间距。
label_opts	标签配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

	仪表盘也被称为拨号图表或速度表图。其显示类似于拨号/速度计上的读数的数据，是一种拟物化的展示形式。仪表盘的颜色可以用来划分指示值的类别，使用刻度标示数据，指针指示维度，指针角度表示数值。
	仪表盘只需分配最小值和最大值，并定义一个颜色范围，指针（指数）将显示出关键指标的数据或当前进度。仪表盘可用于许多目的，如速度、体积、温度、进度、完成率、满意度等。

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
data_pair	系列数据项，格式为 [(key1, value1), (key2, value2)]。
is_selected	是否选中图例。
min_	最小的数据值。
max_	最大的数据值。
split_number	仪表盘平均分割段数。
start_angle	仪表盘起始角度。圆心 正右手侧为0度，正上方为 90 度，正左手侧为 180 度。
end_angle	仪表盘结束角度。
label_opts	标签配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

	环形图是由两个及两个以上大小不一的饼图叠在一起，挖去中间的部分所构成的图形。环形图与饼图类似，但是又有区别。环形图中间有一个“空洞”，每个样本用一个环来表示，样本中的每一部分数据用环中的一段表示。因此环形图可显示多个样本各部分所占的相应比例，从而有利于构成的比较研究。

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
data_pair	系列数据项，格式为 [(key1, value1), (key2, value2)]。
color	系列 label 颜色。
radius	饼图的半径，数组的第一项是内半径，第二项是外半径默认设置成百分比。
center	饼图的中心（圆心）坐标，数组的第一项是横坐标，第二项是纵坐标默认设置成百分比。
rosetype	是否展示成南丁格尔图，通过半径区分数据大小，有'radius'和'area'两种模式。
is_clockwise	饼图的扇区是否是顺时针排布。
label_opts	标签配置项。
tooltip_opts	提示框组件配置项。
itemstyle_opts	图元样式配置项。
```

	雷达图又被叫做蜘蛛网图，适用于显示三个或更多的维度的变量。雷达图是以在同一点开始的轴上显示的三个或更多个变量的二维图表的形式来显示多元数据的方法，其中轴的相对位置和角度通常是无意义的。
	雷达图的每个变量都有一个从中心向外发射的轴线，所有的轴之间的夹角相等，同时每个轴有相同的刻度，将轴到轴的刻度用网格线链接作为辅助元素，连接每个变量在其各自的轴线的数据点成一条多边形。

```python
属性	说明
schema	雷达指示器配置项列表。
shape	雷达图绘制类型，可选 'polygon' 和 'circle'。
textstyle_opts	文字样式配置项。
splitline_opt	分割线配置项。
splitarea_opt	分隔区域配置项。
axisline_opt	坐标轴轴线配置项。
```

```python
属性	说明
series_name	系列名称，用于 tooltip 的显示，legend 的图例筛选。
data	系列数据项。
is_selected	是否选中图例。
symbol	ECharts 提供的标记类型。
color	系列 label 颜色。
label_opts	标签配置项。
linestyle_opts	线样式配置项。
areastyle_opts	区域填充样式配置项。
tooltip_opts	提示框组件配置项。
```

```python
属性	说明
name	指示器名称。
min_	指示器的最小值，可选，默认为0。
max_	指示器的最大值，可选，建议设置。
color	标签特定的颜色。
	
```
相关资料：Python数据可视化之Matplotlib与Pyecharts.pptx
[https://download.csdn.net/download/qq_45832050/13086496](https://download.csdn.net/download/qq_45832050/13086496)

