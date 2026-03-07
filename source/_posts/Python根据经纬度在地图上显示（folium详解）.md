---
title: Python根据经纬度在地图上显示（folium详解）
date: 2026-01-02 08:08:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Python根据经纬度在地图上显示（folium）)

# 一、Folium详解
Folium 是一个用于在 Python 中创建交互式地图的库。它基于 Leaflet.js 构建，可以轻松地将地图嵌入到 Jupyter Notebook 或 Web 应用中。以下是 Folium 的一些主要特点和使用方法：

**主要特点**
* 交互式地图：支持缩放、平移等交互操作。 
* 多种图层：支持多种地图图层，如 OpenStreetMap、Stamen、CartoDB 等。 
* 标记和弹出窗口：可以在地图上添加标记和弹出窗口，显示详细信息。 
* 矢量图层：支持绘制线、多边形等矢量图层。 
* 热力图：支持创建热力图，用于可视化数据密度。 
* 集成 GeoJSON：支持加载和显示 GeoJSON 数据。

## folium.Map参数简要介绍
1、location地图中心点 经纬度，list 或者 tuple 格式，顺序为 latitude(纬度), longitude(经度)

2、zoom_start地图等级 缩放值，默认为 10，值越大比例尺越小，地图放大级别越大

3、tiles 显示样式，默认*‘OpenStreetMap'*，也就是开启街道显示；也有一些其他的内建地图样式，如'Stamen  Terrain'、'Stamen Toner'、'Mapbox Bright'、'Mapbox Control Room'等；也可以传入'None'来绘制一个没有风格的朴素地图，或传入一个URL来使用其它的自选osm

4、crs 地理坐标参考系统，默认为"EPSG3857"

5、width：int型或str型，int型时，传入的是地图宽度的像素值；str型时，传入的是地图宽度的百分比，形式为'xx%'。默认为'100%'

6、height：控制地图的高度，格式同width

7、max_zoom：int型，控制地图可以放大程度的上限，默认为18

8、attr：str型，当在tiles中使用自选URL内的osm时使用，用于给自选osm命名

9、control_scale：bool型，控制是否在地图上添加比例尺，默认为False即不添加

10、no_touch：bool型，控制地图是否禁止接受来自设备的触控事件譬如拖拽等，默认为False，即不禁止

## folium.TileLayer参数简要介绍
folium.TileLayer 是 Folium 库中的一个类，用于在地图上添加不同的图层。这些图层通常是从在线地图服务提供商（如 OpenStreetMap、Stamen、CartoDB 等）获取的瓦片图层。通过使用 folium.TileLayer，你可以轻松地在地图上切换不同的底图，以满足不同的需求。

1、tiles: (str, default 'OpenStreetMap') - 图层的名称或 URL 模板。常见的图层名称包括 'OpenStreetMap', 'Stamen Terrain', 'Stamen Toner', 'Stamen Watercolor', 'CartoDB positron', 'CartoDB dark_matter' 等。

2、name: (str, default None) - 图层的名称，用于图层控制。

3、overlay: (bool, default False) - 是否将此图层作为覆盖层。如果设置为 True，则图层不会默认显示，需要通过图层控制手动启用。

4、control: (bool, default True) - 是否将此图层添加到图层控制中。

5、show: (bool, default True) - 是否在初始化时显示此图层。

6、attr: (str, default None) - 图层的归属信息，通常用于显示版权信息。

7、subdomains: (str or list, default 'abc') - 子域名列表，用于负载均衡。

8、min_zoom: (int, default 0) - 最小缩放级别。

9、max_zoom: (int, default 18) - 最大缩放级别。

10、min_lat: (float, default -Infinity) - 最小纬度。

11、max_lat: (float, default Infinity) - 最大纬度。

12、min_lon: (float, default -Infinity) - 最小经度。

13、max_lon: (float, default Infinity) - 最大经度。

14、no_wrap: (bool, default False) - 是否禁用经度环绕。

15、tms: (bool, default False) - 是否使用 TMS 瓦片顺序。

16、opacity: (float, default 1) - 图层的透明度，范围从 0 到 1。

17、zindex: (int, default 1) - 图层的 z-index。

18、detect_retina: (bool, default False) - 是否检测 Retina 屏幕并使用高分辨率瓦片。

19、tile_size: (int, default 256) - 瓦片的大小（像素）。

## folium.FeatureGroup参数简要介绍
folium.FeatureGroup 是 Folium 库中的一个类，用于将多个地图元素（如标记、线、多边形等）组合在一起，形成一个逻辑组。通过使用 FeatureGroup，你可以更方便地管理和控制这些元素，例如一次性显示或隐藏整个组，或者为组内的所有元素添加相同的样式或事件处理程序。

**主要用途**
* 分组管理：将相关的地图元素分组，便于管理和控制。 
* 图层控制：通过图层控制面板，可以方便地显示或隐藏整个组。 
* 样式统一：可以为组内的所有元素统一设置样式。 
* 事件处理：可以为组内的所有元素统一添加事件处理程序。

**主要参数**
1、name: (str, default None) - 组的名称，用于图层控制。

2、overlay: (bool, default True) - 是否将此组作为覆盖层。如果设置为 True，则组不会默认显示，需要通过图层控制手动启用。

3、control: (bool, default True) - 是否将此组添加到图层控制中。

4、show: (bool, default True) - 是否在初始化时显示此组。

## folium.MarkerCluster参数简要介绍
folium.MarkerCluster 是 Folium 库中的一个类，用于将多个标记（markers）聚合成一个集群（cluster）。当地图缩放级别较高时，标记会聚集在一起形成一个聚合点，从而避免标记重叠，提高地图的可读性和性能。当用户放大地图时，聚合点会逐渐分解成单个标记。

**主要用途**
* 标记聚合：将大量标记聚合成一个或多个聚合点，避免标记重叠。 
* 提高性能：减少地图上显示的标记数量，提高地图的加载和渲染速度。 
* 增强用户体验：用户可以通过缩放地图来查看不同级别的细节。

**主要参数**
1、name: (str, default None) - 集群的名称，用于图层控制。

2、overlay: (bool, default True) - 是否将此集群作为覆盖层。如果设置为 True，则集群不会默认显示，需要通过图层控制手动启用。

3、control: (bool, default True) - 是否将此集群添加到图层控制中。

4、show: (bool, default True) - 是否在初始化时显示此集群。

5、icon_create_function: (str, default None) - 自定义聚合图标创建函数，用于控制聚合点的外观。

6、spiderfy_on_max_zoom: (bool, default True) - 在最大缩放级别时是否展开聚合点。

7、disable_clustering_at_zoom: (int, default None) - 在指定的缩放级别以上禁用聚合。

8、max_cluster_radius: (int, default 80) - 聚合点的最大半径（像素）。

9、chunked_loading: (bool, default False) - 是否启用分块加载，适用于大量标记。

10、chunk_progress: (str, default None) - 分块加载的进度条样式。

11、polygon_options: (dict, default None) - 聚合点的多边形选项。

## folium.Marker参数简要介绍
folium.Marker 是 Folium 库中的一个类，用于在地图上添加标记（marker）。标记可以用来表示特定的位置，并且可以附加弹出窗口（popup）来显示更多信息。通过 folium.Marker，你可以轻松地在地图上添加各种类型的标记，并对其进行自定义。

**主要用途**
* 位置标记：在地图上标记特定的位置。 
* 信息显示：通过弹出窗口显示与标记相关的信息。 
* 自定义图标：使用自定义图标来表示标记。 
* 事件处理：为标记添加点击、鼠标悬停等事件处理程序。

**主要参数**

1、location: (list or tuple, required) - 标记的位置，格式为 [latitude, longitude]。

2、popup: (str or folium.Popup, default None) - 标记的弹出窗口内容。

3、icon: (folium.Icon, default None) - 标记的图标。

4、draggable: (bool, default False) - 标记是否可拖动。

5、tooltip: (str or folium.Tooltip, default None) - 标记的工具提示内容。

6、z_index_offset: (int, default 0) - 标记的 z-index 偏移量。

7、opacity: (float, default 1) - 标记的透明度，范围从 0 到 1。

8、rise_on_hover: (bool, default False) - 当鼠标悬停时是否提升标记的层级。

9、rise_offset: (int, default 250) - 标记提升的层级偏移量。

10、rotation_angle: (int, default 0) - 标记的旋转角度。

11、rotation_origin: (str, default 'center bottom') - 标记的旋转原点。

## folium.Popup参数简要介绍
folium.Popup 是 Folium 库中的一个类，用于在地图上的标记或其他元素上显示弹出窗口。弹出窗口可以包含文本、HTML 内容、图片等，为用户提供更多的信息。通过 folium.Popup，你可以轻松地为地图上的标记、线、多边形等元素添加丰富的交互内容。

**主要用途**
* 信息显示：在标记或其他元素上显示详细信息。 
* 富媒体内容：支持文本、HTML、图片等富媒体内容。 
* 自定义样式：通过 CSS 自定义弹出窗口的样式。 
* 事件处理：为弹出窗口添加点击、关闭等事件处理程序。

**主要参数**
1、html: (str, required) - 弹出窗口的内容，可以是纯文本或 HTML 字符串。

2、max_width: (int, default 300) - 弹出窗口的最大宽度（像素）。

3、min_width: (int, default 50) - 弹出窗口的最小宽度（像素）。

4、max_height: (int, default None) - 弹出窗口的最大高度（像素），如果设置为 None，则没有最大高度限制。

5、show: (bool, default False) - 是否在初始化时自动显示弹出窗口。

6、sticky: (bool, default False) - 是否使弹出窗口在地图移动时保持固定位置。

7、no_wrap: (bool, default False) - 是否禁用文本换行。

8、parse_html: (bool, default False) - 是否解析 HTML 内容。

## folium.Icon参数简要介绍
folium.Icon 是 Folium 库中的一个类，用于自定义地图上标记（marker）的图标。通过 folium.Icon，你可以改变标记的图标样式、颜色、大小等属性，使其更符合你的需求。这使得地图上的标记更加多样化和个性化。

**主要用途**
* 图标样式：改变标记的图标样式。 
* 颜色定制：改变标记的颜色。 
* 图标大小：调整标记的大小。 
* 图标旋转：旋转标记图标。 
* 图标阴影：自定义标记的阴影效果。

**主要参数**
1、icon_color: (str, default 'blue') - 图标颜色，可选值包括 'white', 'red', 'blue', 'green', 'orange', 'purple', 'pink', 'cadetblue', 'gray', 'darkred', 'lightred', 'beige', 'darkblue', 'lightblue', 'darkgreen', 'lightgreen', 'darkpurple', 'lightpurple', 'black'。

2、color: (str, default 'white') - 图标背景颜色。

3、icon: (str, default 'info-sign') - 图标样式，使用 Font Awesome 图标库中的图标名称。

4、prefix: (str, default 'glyphicon') - 图标前缀，可选值包括 'glyphicon' 和 'fa'（Font Awesome）。

5、icon_anchor: (tuple, default (12, 41)) - 图标的锚点位置，格式为 (x, y)。

6、shadow: (bool, default True) - 是否显示图标阴影。

7、shadow_size: (tuple, default (41, 41)) - 图标阴影的大小，格式为 (width, height)。

8、shadow_anchor: (tuple, default (12, 41)) - 图标阴影的锚点位置，格式为 (x, y)。

9、popup_anchor: (tuple, default (1, -34)) - 弹出窗口的锚点位置，格式为 (x, y)。

10、spin: (bool, default False) - 是否使图标旋转（仅适用于 Font Awesome 图标）。

11、border_color: (str, default None) - 图标的边框颜色。

12、border_width: (int, default 3) - 图标的边框宽度。

## folium.LayerControl参数简要介绍
folium.LayerControl 是 Folium 库中的一个类，用于在地图上添加图层控制面板。图层控制面板允许用户在地图上动态地显示或隐藏不同的图层，从而提供更好的交互体验。通过 folium.LayerControl，你可以轻松地管理多个图层，并让用户根据需要选择显示哪些图层。

**主要用途**
* 图层管理：管理地图上的多个图层。 
* 用户交互：允许用户动态地显示或隐藏图层。 
* 分组图层：将图层分组，方便管理和选择。 
* 自定义样式：自定义图层控制面板的样式。

**主要参数**
1、position: (str, default 'topright') - 图层控制面板的位置，可选值包括 'topleft', 'topright', 'bottomleft', 'bottomright'。

2、collapsed: (bool, default True) - 图层控制面板是否默认折叠。

3、autoZIndex: (bool, default True) - 是否自动为每个图层分配 z-index。

4、hideSingleBase: (bool, default False) - 如果只有一个基础图层，是否隐藏基础图层选择。

5、overlayPosition: (str, default 'bottom') - 叠加图层的位置，可选值包括 'top', 'bottom'。

6、baseLayerPosition: (str, default 'top') - 基础图层的位置，可选值包括 'top', 'bottom'。

7、sortLayers: (bool, default False) - 是否按字母顺序排序图层。

8、sortFunction: (function, default None) - 自定义图层排序函数。

9、namedLayers: (bool, default True) - 是否显示图层名称。

10、collapsedLayers: (list, default []) - 默认折叠的图层列表。

## folium.LatLngPopup参数简要介绍
folium.LatLngPopup 是 Folium 库中的一个类，用于在地图上显示用户点击位置的经纬度坐标。当用户在地图上点击某个位置时，会弹出一个包含该位置经纬度坐标的弹出窗口。这个功能非常有用，特别是在需要获取地图上特定位置的坐标时。

**主要用途**
* 获取坐标：显示用户点击位置的经纬度坐标。
* 用户交互：增强地图的交互性，提供即时反馈。 
* 数据收集：帮助用户快速收集地图上的坐标数据。

**主要参数**
1、`popup: (str, default 'Lat: {lat}<br>Lng: {lng}')` - 弹出窗口的内容模板，其中 {lat} 和 {lng} 会被实际的纬度和经度值替换。

## folium.MousePosition参数简要介绍
folium.MousePosition 是 Folium 库中的一个类，用于在地图上显示鼠标指针当前位置的经纬度坐标。当用户在地图上移动鼠标时，会实时显示鼠标指针所在位置的经纬度坐标。这个功能非常有用，特别是在需要实时获取地图上特定位置的坐标时。
**主要用途**
* 实时坐标显示：显示鼠标指针当前位置的经纬度坐标。 
* 用户交互：增强地图的交互性，提供即时反馈。 
* 数据收集：帮助用户快速收集地图上的坐标数据。

**主要参数**
1、position: (str, default 'bottomright') - 鼠标位置控件的位置，可选值包括 'topleft', 'topright', 'bottomleft', 'bottomright'。

2、separator: (str, default ' | ') - 经度和纬度之间的分隔符。

3、empty_string: (str, default 'Lon: , Lat: ') - 当鼠标不在地图上时显示的字符串。

4、lat_first: (bool, default False) - 是否先显示纬度后显示经度。

5、num_digits: (int, default 5) - 经纬度的小数位数。

6、prefix: (str, default '') - 显示在经纬度前面的前缀。

7、lat_formatter: (function, default None) - 纬度格式化函数。

8、lng_formatter: (function, default None) - 经度格式化函数。

## folium.GeoJson参数简要介绍

folium.GeoJson 是 Folium 库中的一个类，用于在地图上显示 GeoJSON 数据。GeoJSON 是一种基于 JSON 的地理空间数据交换格式，常用于表示点、线、多边形等地理要素。

**主要参数**
1、data: GeoJSON 数据，可以是字典或文件路径。

2、style_function: 用于定义 GeoJSON 对象样式的函数。

3、highlight_function: 用于定义鼠标悬停时 GeoJSON 对象样式的函数。

4、name: 图层名称，用于图层控制。

5、overlay: 是否作为覆盖层，默认为 True。

6、control: 是否在图层控制中显示，默认为 True。

7、show: 是否默认显示图层，默认为 True。

8、smooth_factor: 平滑因子，用于平滑线条，默认为 1.0。

9、tooltip: 提示信息，可以是字符串或 folium.GeoJsonTooltip 对象。

10、popup: 弹出信息，可以是字符串或 folium.GeoJsonPopup 对象。

11、embed: 是否嵌入 GeoJSON 数据，默认为 True。

12、zoom_on_click: 是否在点击时缩放，默认为 True。

13、marker_property: 指定 GeoJSON 特征属性作为标记，默认为 None。

14、marker: 自定义标记，默认为 None。

## folium.DivIcon参数简要介绍

folium.DivIcon 是 Folium 库中的一个类，用于创建自定义的 HTML 标记图标。与默认的圆形图标不同，DivIcon 允许你使用自定义的 HTML 内容来创建标记图标，从而实现更丰富的视觉效果。

**主要参数**
1、icon_size: 图标的大小，以像素为单位，格式为 (width, height)。

2、icon_anchor: 图标在标记位置的锚点，格式为 (x, y)。默认值为 (0, 0)，表示左上角。

3、popup_anchor: 弹出框相对于图标的锚点，格式为 (x, y)。默认值为 (0, 0)。

4、html: 自定义的 HTML 内容，用于创建图标。可以包含任何有效的 HTML 代码。

5、class_name: 图标的 CSS 类名，用于进一步自定义样式。
## folium 中的 add_child, add_to, 和 save 方法简要介绍
在 folium 库中，add_child, add_to, 和 save 是常用的方法，用于构建和保存地图。下面分别对这三个方法进行详细说明。
1. add_child
add_child 方法用于将一个子元素（如标记、图层、控件等）添加到地图或其他父元素中。这个方法通常在内部被其他方法调用，但在某些情况下也可以直接使用。
`parent.add_child(child, name=None, index=None)
`
参数
child: (Element) - 要添加的子元素。
name: (str, optional) - 子元素的名称。
index: (int, optional) - 插入子元素的位置索引。

2. add_to
add_to 方法用于将一个元素添加到地图或其他父元素中。这个方法通常在创建元素时直接调用，更加简洁和常用。
`element.add_to(parent)
`
参数
parent: (Element) - 要添加到的父元素。

3. save
save 方法用于将地图保存为 HTML 文件。保存后的文件可以在浏览器中打开，查看地图及其所有添加的元素。
`map.save(outfile, close_file=True)`
参数
outfile: (str) - 保存的文件路径和名称。
close_file: (bool, default True) - 是否关闭文件对象。

# 二、Python根据经纬度在地图上显示（示例）
![在这里插入图片描述](/img/5fd3f97364fdb977a776f5bafe77a9e0.png)

## 1.经纬度坐标标记
```python
import pandas as pd
import folium
from folium.plugins import MarkerCluster

data = pd.read_excel('******.xlsx')  # 读取文件
data_1 = data[data['类型'] == '***1']
data_2 = data[data['类型'] == '***2']
data_1.reset_index(inplace=True, drop=True)
data_2.reset_index(inplace=True, drop=True)
# print(data_1.head())
# print(data_2.head())
# exit()
m = folium.Map(location=[31.97117, 116.49872],  # 中心点
               zoom_start=8,  # 初始地图等级
               # 腾讯地图瓦片
               tiles='http://rt1.map.gtimg.com/realtimerender?z={z}&x={x}&y={-y}&type=vector&style=6',
               # 默认参数
               attr='default')
flag = False  # 是否使用聚合
if flag:
    # 创建聚合
    marker_cluster = MarkerCluster().add_to(m)
    # for循环添加标记点
    for i in range(len(data_1)):
        folium.Marker(location=[data_1.loc[i, '纬度'], data_1.loc[i, '经度']],  # 坐标用[纬度，经度]
                      popup=folium.Popup(str(data_1.loc[i, 'NAME']),
                                         parse_html=True,
                                         tooltip=str(data_1.loc[i, 'NAME']),
                                         max_width=100),  # 提示语横向完全显示
                      icon=folium.Icon(color='red')
                      ).add_to(marker_cluster)
    for j in range(len(data_2)):
        folium.Marker(location=[data_2.loc[j, '纬度'], data_2.loc[j, '经度']],  # 坐标用[纬度，经度]
                      popup=folium.Popup(str(data_2.loc[j, 'NAME']),
                                         parse_html=True,
                                         tooltip=str(data_2.loc[j, 'NAME']),
                                         max_width=100),  # 提示语横向完全显示
                      icon=folium.Icon(color='blue')
                      ).add_to(marker_cluster)
else:
    # for循环添加标记点
    for i in range(len(data_1)):
        folium.Marker(location=[data_1.loc[i, '纬度'], data_1.loc[i, '经度']],  # 坐标用[纬度，经度]
                      popup=folium.Popup(str(data_1.loc[i, 'NAME']),
                                         parse_html=True,
                                         tooltip=str(data_1.loc[i, 'NAME']),
                                         max_width=100),  # 提示语横向完全显示
                      icon=folium.Icon(color='red'),
                      ).add_to(m)
    for j in range(len(data_2)):
        folium.Marker(location=[data_2.loc[j, '纬度'], data_2.loc[j, '经度']],  # 坐标用[纬度，经度]
                      popup=folium.Popup(str(data_2.loc[j, 'NAME']),
                                         parse_html=True,
                                         tooltip=str(data_2.loc[j, 'NAME']),
                                         max_width=100),  # 提示语横向完全显示
                      icon=folium.Icon(color='blue'),
                      ).add_to(m)
'''为地图对象添加点击显示经纬度的子功能'''
m.add_child(folium.LatLngPopup())
# 点击新增
# m.add_child(folium.ClickForMarker())
m.save('坐标分布图.html')

```
## 2.经纬度坐标分组标记

```python
import pandas as pd
import folium
from folium.plugins import MarkerCluster, MousePosition
from folium import FeatureGroup, LayerControl

# tile = 'http://rt1.map.gtimg.com/realtimerender?z={z}&x={x}&y={-y}&type=vector&style=0'
tile = 'https://webst02.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}'

df = pd.read_excel('******.xlsx')  # 读取文件
df_1 = df[df['类型'] == '***1']
df_2 = df[df['类型'] == '***2']
df_1.reset_index(inplace=True, drop=True)
df_2.reset_index(inplace=True, drop=True)
distriction = df['所属市'].drop_duplicates()

m = folium.Map(location=[30.97117, 132.49872],  # 地图中心点
               tiles=None,
               control_scale=True,  # 显示比例尺
               zoom_start=8)  # 初始等级
folium.TileLayer(tiles=tile, attr='default', name='省').add_to(m)  # 地图瓦片添加命名

# #创建组
for i in distriction:
    exec(str(i) + ' = ' + 'FeatureGroup(name="' + str(i) + '",show=False).add_to(m)')

# 创建聚合
for j in distriction:
    # 是否将临近点聚合
    # exec(str(j) + 'mc = ' + 'MarkerCluster().add_to(' + str(j) + ')')
    exec(str(j) + 'mc = ' + str(j))

# for循环添加标记点
for k in range(len(df_1)):
    exec('''folium.Marker(location=[df_1.loc[k,'纬度'], df_1.loc[k,'经度']],  
                  popup=folium.Popup(str(df_1.loc[k,'NAME']), 
                                     parse_html=True, 
                                     max_width=150),                #提示语横向完全显示
                  icon=folium.Icon(color='red')      
                 ).add_to(''' + str(df_1.loc[k, '所属市']) + 'mc)')
for k in range(len(df_2)):
    exec('''folium.Marker(location=[df_2.loc[k,'纬度'], df_2.loc[k,'经度']],  
                  popup=folium.Popup(str(df_2.loc[k,'NAME']), 
                                     parse_html=True, 
                                     max_width=150),                #提示语横向完全显示 
                  icon=folium.Icon(color='blue')      
                 ).add_to(''' + str(df_2.loc[k, '所属市']) + 'mc)')

LayerControl(collapsed=False).add_to(m)
'''为地图对象添加点击显示经纬度的子功能'''
m.add_child(folium.LatLngPopup())
'''在地图上添加鼠标位置插件'''
MousePosition().add_to(m)
m.save('省市坐标分布图.html')  # 保存到当前目录下

```
![在这里插入图片描述](/img/5c4a1074ea4349139e9573d04c5f14c2.png)

参考：[folium官网](https://python-visualization.github.io/folium/latest/reference.html)

# 三、常用地图瓦片源地址

## 1.高德地图（火星坐标）

高德矢量底图

```bash
https://webrd04.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=7&x={x}&y={y}&z={z}
```

![在这里插入图片描述](/img/6644bf94cc994496880299bb0b1b7758.png)

高德卫星影像

```bash
https://webst01.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}
```

![在这里插入图片描述](/img/22c633c5e6f944b09671eabcd6bfe123.png)


高德路网注记

```bash
https://webst01.is.autonavi.com/appmaptile?style=8&x={x}&y={y}&z={z}
```

![在这里插入图片描述](/img/1f3488f4b8dc4b1eb7528b3123057c89.png)


* style=6：
这个参数指定了地图的样式。不同的样式值会返回不同类型的地图瓦片。例如，style=6 可能表示某种特定的地图风格，如标准地图、卫星地图等。
* x={x}：
这是瓦片的 X 坐标。在瓦片地图中，地球表面被划分为多个瓦片，每个瓦片都有一个唯一的 (x, y) 坐标。
* y={y}：
这是瓦片的 Y 坐标。
* z={z}：
这是地图的缩放级别（Zoom Level）。缩放级别决定了地图的详细程度，数值越大，地图越详细。

## 2.ArcGIS图源（84坐标）

ArcGIS卫星影像

```bash
https://server.arcgisonline.com/arcgis/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}.png
```

![在这里插入图片描述](/img/d10bba3510e948069058a5761826c24f.png)


ArcGIS街道

```bash
https://server.arcgisonline.com/arcgis/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}.png
```

![在这里插入图片描述](/img/187690524dc549f489c570f3576763a5.png)


实时

```bash
https://wayback.maptiles.arcgis.com/arcgis/rest/services/world_imagery/wmts/1.0.0/default028mm/mapserver/tile/45441/{z}/{y}/{x}
```

![在这里插入图片描述](/img/5caa36b41f6442bb83467c62e194ce8d.png)
# 四、常用各省GeoJSON数据

```bash
https://geo.datav.aliyun.com/areas_v3/bound/370000_full.json
```

37000：行政区代码

## 常见省份代码

```bash
北京市: 110000
天津市: 120000
河北省: 130000
山西省: 140000
内蒙古自治区: 150000
辽宁省: 210000
吉林省: 220000
黑龙江省: 230000
上海市: 310000
江苏省: 320000
浙江省: 330000
安徽省: 340000
福建省: 350000
江西省: 360000
山东省: 370000
河南省: 410000
湖北省: 420000
湖南省: 430000
广东省: 440000
广西壮族自治区: 450000
海南省: 460000
重庆市: 500000
四川省: 510000
贵州省: 520000
云南省: 530000
西藏自治区: 540000
陕西省: 610000
甘肃省: 620000
青海省: 630000
宁夏回族自治区: 640000
新疆维吾尔自治区: 650000
```

## 按省并显示各地市名称

```bash
import folium
import requests
import pandas as pd
from folium import FeatureGroup, LayerControl
from folium.plugins import MarkerCluster, MousePosition

# 设置地图瓦片
# tile = 'http://rt1.map.gtimg.com/realtimerender?z={z}&x={x}&y={-y}&type=vector&style=0'
# tile = 'https://wayback.maptiles.arcgis.com/arcgis/rest/services/world_imagery/wmts/1.0.0/default028mm/mapserver/tile/45441/{z}/{y}/{x}'
tile = 'https://webst02.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}'  # 卫星影像图

# 读取数据
df = pd.read_excel(r'./******.xlsx')  # 读取文件

# 过滤数据
df_fd = df[df['类型'] == '***']
df_gf = df[df['类型'] == '***']
df_fd.reset_index(inplace=True, drop=True)
df_gf.reset_index(inplace=True, drop=True)

# 计算地图中心点
lat_min, lat_max = df['纬度'].min(), df['纬度'].max()
lon_min, lon_max = df['经度'].min(), df['经度'].max()
lat, lon = (lat_min + lat_max) / 2, (lon_min + lon_max) / 2

# 获取**省的 GeoJSON 数据
url = "https://geo.datav.aliyun.com/areas_v3/bound/******_full.json"
response = requests.get(url)
geojson_data = response.json()

# 创建地图对象
m = folium.Map(location=[lat, lon],  # 地图中心点
               tiles=None,
               control_scale=True,  # 显示比例尺
               zoom_start=7)  # 初始等级
folium.TileLayer(tiles=tile, attr='QZK', name='省份').add_to(m)  # 地图瓦片添加命名

# 添加地市名称
folium.GeoJson(geojson_data, name='地市边界').add_to(m)

# 直接显示地市名称
for feature in geojson_data['features']:
    city_name = feature['properties']['name']
    centroid = feature['properties']['center']
    folium.Marker(
        location=[centroid[1], centroid[0]],
        icon=folium.DivIcon(html=f'<div style="font-size: 12pt; color: black">{city_name}</div>'),
        tooltip=city_name
    ).add_to(m)

# 创建特征组
distriction = df['***'].drop_duplicates()
for i in distriction:
    exec(str(i) + ' = ' + 'FeatureGroup(name="' + str(i) + '",show=False).add_to(m)')

# 创建聚合
for j in distriction:
    # 是否将临近点聚合
    # exec(str(j) + 'mc = ' + 'MarkerCluster().add_to(' + str(j) + ')')
    exec(str(j) + 'mc = ' + str(j))

# for循环添加标记点
for k in range(len(df_fd)):
    name2code2cap = str(df_fd.loc[k,'NAME']) + " " + str(df_fd.loc[k,'**'])  # str(df_fd.loc[k,'**']) + " " +
    exec('''folium.Marker(location=[df_fd.loc[k,'纬度'], df_fd.loc[k,'经度']],
                  popup=folium.Popup(name2code2cap,
                                     parse_html=True,
                                     max_width=150),                #提示语横向完全显示
                  icon=folium.Icon(color='red')
                 ).add_to(''' + str(df_fd.loc[k, '***']) + 'mc)')
for k in range(len(df_gf)):
    name2code2cap = str(df_gf.loc[k, 'NAME']) + " " + str(df_gf.loc[k, '**'])  # str(df_gf.loc[k, '**']) + " " +
    exec('''folium.Marker(location=[df_gf.loc[k,'纬度'], df_gf.loc[k,'经度']],
                  popup=folium.Popup(name2code2cap,
                                     parse_html=True,
                                     max_width=150),                #提示语横向完全显示
                  icon=folium.Icon(color='blue')
                 ).add_to(''' + str(df_gf.loc[k, '***']) + 'mc)')

LayerControl(collapsed=False).add_to(m)
'''为地图对象添加点击显示经纬度的子功能'''
m.add_child(folium.LatLngPopup())
'''在地图上添加鼠标位置插件'''
MousePosition().add_to(m)
m.save('./坐标分布图.html')  # 保存到当前目录下
```
![在这里插入图片描述](/img/3581899a30ad475ba0de2f33d9ab011b.png)

