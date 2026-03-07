---
title: AGV分拣工业场景Baseline
date: 2026-01-01 08:02:00
tags:
  - "数学建模"
  - "AGV"
  - "物流"
  - "工业场景"
  - "竞赛"
categories:
  - "数学建模"
---

﻿@[TOC](AGV分拣工业场景Baseline)
![在这里插入图片描述](/img/4bd8dc94a6a1416dafc13b1a151ddd61.png)

# 2025西门子Xcelerator公开赛 MioVerse赛道 JCIIOT开发者大赛 赛题与规则

某快递公司需要优化某一个自动分拣单元的任务分配与AGV（自动导引车）路线规划，以实现快件从供件台到目的流向格口的高效运输。AGV需以最短时间完成运输任务，同时避免碰撞并满足各项约束条件。

## 题目描述

给定以下信息：

1.  **供件台**：6个，快件的到达点位，AGV到供件台取料点取件，编号为 ["Tiger", "Dragon", "Horse", "Rabbit", "Ox", "Monkey"]，每个供件台有一个取料点位。
2.  **格口**：16个，快件的目的地流向对应的格口，编号为 ["Beijing", "Shanghai", "Suzhou", ..., "Chongqing"]，每个格口有相邻的4个卸料点位。
3.  **AGV**：12辆，均处于可用状态（可以仅使用部分AGV），编号为 ["Optimus", "Bumblebee", ..., "Jazz"]。实现快件从供件台取料点到格口卸料点的运输，到达目的格口对应的卸料点位后，自动倾倒快件进入格口。
4.  **AGV 属性**：
    *   尺寸：40cm × 40cm × 30cm。
    *   速度：1m/s，不考虑加、减速度影响。
    *   转向：只能原地旋转转向，转向角度为90度整数倍，每次转向耗时1秒。
    *   取货/卸货：每次均耗时1秒。
    *   原地等待：可以采取AGV原地静止用于避让，每次等待时间为1s的整数倍。
    *   初始状态：每个AGV的初始坐标 [x, y] 和朝向pitch（角度，单位：度，定义X轴正向为0度，Y轴正向为90度，X轴负向为180度，Y轴负向为270度）。
5.  **区域地图、供件台、流向格口坐标**：
    *   区域地图由20m×20m大小的区域组成，每个方格尺寸长宽均为1米；AGV仅可以在沿着方格中心移动。
    *   地图中包含供件台（蓝色）、目的地格口（绿色）和AGV初始位置（橙色）的示意布局。
    *   示例：AGV Optimus初始坐标（3,1），朝向为90度。
6.  **快件序列**：与格口流向匹配的随机序列，包含导入台编号、目的流向格口编号、时效类型（高优先级紧急件、普通件）、高优先级快件剩余有效时间。

## 约束条件

1.  每个AGV每次只能运输一个快件。
2.  格口有货物，AGV才能成功取得，且每个供件台1秒只能完成一次AGV取料。
3.  AGV在导入台（起始点）的左侧或右侧方格唯一取货点取货（例如Horse取料点坐标为（2,14）；Monkey取料点坐标为（19,14））。AGV在格口（目的地）对应的4个卸料点位均可以执行卸货操作（卸料点位于目的地相邻的4个方格，如目的地Hangzhou坐标为（6,16），其对应的卸料点位坐标有（6,15）、（6,17）、（5,16）、（7,16））。AGV不可以越过20m×20m的地图范围。
4.  只能按照每个供件台给定的快件序列取快件。
5.  高优先级快件必须在指定剩余时间内运输到目的地（约束满足额外奖励10分，否则被扣分5分）；时效类型（高优先级紧急件，普通件）紧急件含义：部分晚到的快件，需要赶发车班次，若无法在规定时间内完成会导致终端用户收货“延迟”。
6.  快件必须在目的地格口指定的卸料点位卸货。
7.  AGV只能沿x轴和y轴方向移动，每秒一格，不能斜向移动。
8.  除供件台和格口位置之外，所有空白区域均可以AGV通行，当AGV在同一时刻出现在同一位置、或者相向运动时交换位置视为碰撞发生。
9.  AGV原地转向只能正负90度，和180度转向，任何角度的转向均需1秒完成，取放料的同一时刻不可转向。

## 竞赛规则

*   **输入简介（地图与AGV状态信息参考附件1；赛题任务样例参考附件2）随机生成100个快件任务数据，其中有2个为Urgent 高优先级任务。**：
    *   task_id：任务编号
    *   start_point：起始点-导入台编号
    *   end_point：终点-目的流向格口编号
    *   priority：时效类型（Normal，Urgent ）
    *   remaining_time：Urgent 高优先级快件剩余有效时间
*   **算法与要求：强调目标（考虑紧急任务、全局任务协同效率）+创新与AI结合**：
    *   特别鼓励参赛选手发挥创新思维，运用先进的AI/ LLM技术，探索出更具创新性、高效性的解决方案。
*   **输出与格式要求：（详细内容参考附件4：输出格式要求）**：
参赛者需要编写python程序，该python 程序可以生成符合要求的csv格式的文件。违反规则的输出结果计为零分。生成的csv 需符合下列格式，具体样例见样表。
    *   Timestamp: 需覆盖从0时刻到全部任务完成时刻。
    * Name：AGV名称，保持大小写一致
    * X：X轴坐标值
    * Y：Y轴坐标值
    * Pitch：AGV朝向角度
    * Loaded：当前是否载货，要求格式为小写的“true”或“false”
    * Destination：当前载货目的地，非载货时为空“”
    * Emergency：当前载货是否紧急任务，要求格式为小写的“true”或“false”

## 评分规则

本次评审满分100分，将结合选手提交的AGV轨迹信息.csv文件、解决方案说明文档进行综合评价。AGV轨迹信息.csv文件通过智能评测程序进行打分，方案设计说明部分将通过专家评审进行打分。综合得分=客观评测程序得分（60%）+专家评审得分(40%)。

### （1）初赛：智能评测程序打分规则

满分120分（含额外附加分20分）
*   **计分规则**：在5分钟内，统计被运达正确流向的所有快件，一个快件计1分，违反约束的任务不得分。
*   **高优先级任务奖罚**：高优先级快件在指定剩余时间内卸货到目的格口，额外奖励10分/任务，若高时效任务未在指定时间内完成，扣5分/任务。
*   **AGV碰撞惩罚**：如果AGV发生碰撞，扣10分，AGV消失，且不可继续使用，快件任务被丢失，不可继续执行。
*   **平分比较**：如果平分，比较最后一个快件任务完成的时间，时间靠前者胜出；若仍旧出现多组参赛者平分，则通过额外相同的多轮随机任务输入，综合考虑评测得分、任务完成时间、程序运行时间等排序。

### （2）复赛：专家主观评审打分考虑因素

满分100分，评价指标与权重如下：

| 评价指标             | 评价要点                        | 权重 |
| :------------------- | :------------------------------ | :--- |
| **AI/LLM 技术应用**  | - LLM在系统中的融合深度与创新性 | 40%  |
|                      | - AI技术的应用场景合理性        |      |
|                      | - 人工智能解决方案的实效性      |      |
|                      | - 智能决策的可解释性            |      |
| **算法创新**         | - 相比传统方法的突破性改进      | 30%  |
|                      | - 算法思路的原创性              |      |
|                      | - 技术路径的创新程度            |      |
| **实用性与可扩展性** | - 实际场景适应能力              | 20%  |
|                      | - 系统扩展难易程度              |      |
|                      | - 部署和维护成本                |      |
|                      | - 计算开销                      |      |
| **代码规范性**       | - 代码规范性                    | 10%  |
|                      | - 可维护性                      |      |

## 潜在 AI/LLM 应用思路启发

### （1）基于大语言模型的多智能体 AGV 协同

利用多个智能体合作完成任务调度与路径规划，如一个调度Agent，负责全局任务分配；每个AGV是一个独立的agent，独立决策单元，处理任务接受与路径规划，判断是否需要接受任务，以及找到最优路径；LLM负责智能体间的信息传递与决策支持等。融合多智能体系系统、大语言模型和智能算法开发的综合应用。

### （2）基于 AI/LLM 的 AIGC 功能辅助算法开发

该框架覆盖从需求分析、算法设计到实现测试的全过程。通过LLM的协助，开发者可以更准确地理解问题需求，获得算法设计建议，并在代码实现过程中得到持续的优化支持。这种人机协作的开发模式不仅提高了开发效率，还能够充分利用LLM的知识库，为方案设计提供更多创新思路。例如通过LLM理解赛题需求、实现算法设计、算法测试与优化、算法解决方案输出；最后辅助参赛选手优化输出能够解决该问题的算法代码。

# 数据分析（此版本无法保障0碰撞）
## 附件1——agv_position.csv

```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# 读取CSV文件数据
df = pd.read_csv(r'./赛题附件-输入输出格式说明/附件1——map_data.csv')

# 分离不同类型的数据点
start_points = df[df['type'] == 'start_point']
end_points = df[df['type'] == 'end_point']
agv_points = df[df['type'] == 'agv']

# 创建图形
plt.figure(figsize=(14, 10))

# 绘制开始点（蓝色）
for i, row in start_points.iterrows():
    plt.scatter(row['x'], row['y'], color='blue', s=100, alpha=0.7)
    plt.text(row['x'] + 0.2, row['y'] + 0.2, row['name'], fontsize=9,
             ha='left', va='bottom', color='blue')

# 绘制结束点（红色）
for i, row in end_points.iterrows():
    plt.scatter(row['x'], row['y'], color='red', s=100, alpha=0.7)
    plt.text(row['x'] + 0.2, row['y'] + 0.2, row['name'], fontsize=9,
             ha='left', va='bottom', color='red')

# 绘制AGV小车（绿色箭头）
for i, row in agv_points.iterrows():
    x, y, angle = row['x'], row['y'], row['pitch']

    # 将角度转换为弧度
    rad = np.deg2rad(angle)
    # 计算箭头方向（长度设为0.8）
    dx = 0.8 * np.cos(rad)
    dy = 0.8 * np.sin(rad)

    # 绘制箭头
    plt.arrow(x, y, dx, dy,
              head_width=0.5, head_length=0.5,
              fc='green', ec='darkgreen',
              width=0.2, alpha=0.8)

    # 绘制小车位置点
    plt.scatter(x, y, color='green', s=100, alpha=0.7)

    # 添加AGV名称标签（根据朝向调整标签位置）
    if angle == 90:  # 朝上的AGV
        plt.text(x, y - 1.0, row['name'], fontsize=9,
                 ha='center', va='top', color='darkgreen')
    else:  # 朝下的AGV
        plt.text(x, y + 1.0, row['name'], fontsize=9,
                 ha='center', va='bottom', color='darkgreen')

# 添加图例
plt.scatter([], [], color='blue', s=100, label='Start Points')
plt.scatter([], [], color='red', s=100, label='End Points')
plt.scatter([], [], color='green', s=100, label='AGV Positions')
plt.arrow(0, 0, 0, 0, color='green', head_width=0.3, head_length=0.4, label='AGV Direction')
plt.legend(loc='best', fontsize=10)

# 设置坐标轴范围和标签
plt.xlim(0, 22)
plt.ylim(0, 22)
plt.xlabel('X Coordinate', fontsize=12)
plt.ylabel('Y Coordinate', fontsize=12)
plt.title('AGV System Layout Map', fontsize=14)

# 添加网格线
plt.grid(True, linestyle='--', alpha=0.7)

# 添加坐标说明
plt.text(0.5, 21.5, 'AGV Orientation: 90° = Up, 270° = Down',
         fontsize=10, color='gray', ha='left')

# 保存图片
plt.savefig('./visualization/agv_system_layout.png', dpi=300, bbox_inches='tight')
print("布局图已保存为 'agv_system_layout.png'")

# 显示图形
plt.tight_layout()
plt.show()
```
![agv_system_layout.png](/img/76d6b3b7e3d74d3395a588d54d48b53d.png)
## 附件2——agv_task.csv

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.simplefilter('ignore')

# 设置中文字体支持
plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False    # 用来正常显示负号

# 读取任务数据
df = pd.read_csv(r'./赛题附件-输入输出格式说明/附件2——task_csv.csv')

# 1. 绘制各起始点的任务数量分布
plt.figure(figsize=(12, 6))
ax1 = sns.countplot(x='start_point', data=df, palette='viridis')
plt.title('任务数量分布（按起始点）', fontsize=14)
plt.xlabel('起始点', fontsize=12)
plt.ylabel('任务数量', fontsize=12)
plt.xticks(rotation=45)

# 添加数量标签
for p in ax1.patches:
    ax1.annotate(f'{p.get_height()}',
                 (p.get_x() + p.get_width() / 2., p.get_height()),
                 ha='center', va='center',
                 xytext=(0, 5),
                 textcoords='offset points')
plt.tight_layout()
plt.savefig('./visualization/start_point_distribution.png', dpi=300)
plt.show()

# 2. 绘制各终点的任务数量分布（只显示前15个）
plt.figure(figsize=(14, 7))
end_counts = df['end_point'].value_counts().head(15)
ax2 = sns.barplot(x=end_counts.index, y=end_counts.values, palette='coolwarm')
plt.title('任务数量分布（按终点）', fontsize=14)
plt.xlabel('终点', fontsize=12)
plt.ylabel('任务数量', fontsize=12)
plt.xticks(rotation=45)

# 添加数量标签
for i, v in enumerate(end_counts.values):
    ax2.text(i, v + 0.2, str(v), ha='center', va='bottom')

plt.tight_layout()
plt.savefig('./visualization/end_point_distribution.png', dpi=300)
plt.show()

# 3. 绘制优先级分布和紧急任务详情
urgent_tasks = df[df['priority'] == 'Urgent']
normal_count = len(df[df['priority'] == 'Normal'])
urgent_count = len(urgent_tasks)

# 创建双子图
fig, (ax3, ax4) = plt.subplots(1, 2, figsize=(16, 6))

# 优先级分布饼图
priority_counts = [normal_count, urgent_count]
labels = ['普通任务', '紧急任务']
colors = ['#66b3ff', '#ff9999']
explode = (0.05, 0.1)  # 突出显示紧急任务

ax3.pie(priority_counts, explode=explode, labels=labels, colors=colors,
        autopct='%1.1f%%', shadow=True, startangle=90)
ax3.set_title('任务优先级分布', fontsize=14)
ax3.axis('equal')  # 保证饼图是圆形

# 紧急任务详情表格
if not urgent_tasks.empty:
    urgent_data = urgent_tasks[['task_id', 'start_point', 'end_point', 'remaining_time']]
    cell_text = []
    for _, row in urgent_data.iterrows():
        cell_text.append([row['task_id'], row['start_point'], row['end_point'], row['remaining_time']])

    table = ax4.table(cellText=cell_text,
                      colLabels=['任务ID', '起始点', '终点', '剩余时间(秒)'],
                      loc='center',
                      cellLoc='center')
    table.auto_set_font_size(False)
    table.set_fontsize(10)
    table.scale(1, 1.5)
    ax4.axis('off')
    ax4.set_title('紧急任务详情', fontsize=14, pad=20)
else:
    ax4.text(0.5, 0.5, '无紧急任务', ha='center', va='center', fontsize=14)
    ax4.axis('off')

plt.tight_layout()
plt.savefig('./visualization/priority_distribution.png', dpi=300)
plt.show()

# 4. 任务流向热力图（起始点->终点）
plt.figure(figsize=(14, 10))
flow_matrix = pd.crosstab(df['start_point'], df['end_point'])
sns.heatmap(flow_matrix, cmap='YlGnBu', annot=True, fmt='d', linewidths=.5)
plt.title('任务流向热力图（起始点->终点）', fontsize=14)
plt.xlabel('终点', fontsize=12)
plt.ylabel('起始点', fontsize=12)
plt.xticks(rotation=45)
plt.yticks(rotation=0)
plt.tight_layout()
plt.savefig('./visualization/task_flow_heatmap.png', dpi=300)
plt.show()
```
![start_point_distribution.png](/img/7e1626090fae47a998972dd316ae21ff.png)
![end_point_distribution.png](/img/1efc37b8523d4a12906052f6465eb3c6.png)
![priority_distribution.png](/img/7cca5a33144a4a77acde3fb5d141f347.png)
![task_flow_heatmap.png](/img/7075317872334e91996e0bba34c6afa5.png)
## 附件4——agv_trajectory.csv（agv分拣动态模拟）

```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np
from matplotlib.patches import Circle, Wedge
from matplotlib.lines import Line2D
import warnings
warnings.simplefilter('ignore')

# 设置中文字体支持
plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False    # 用来正常显示负号

# 读取地图数据
df_map = pd.read_csv("./赛题附件-输入输出格式说明/附件1——map_data.csv")
start_points = df_map[df_map['type'] == 'start_point']
end_points = df_map[df_map['type'] == 'end_point']
agv_init = df_map[df_map['type'] == 'agv']

# 读取AGV运动数据
# df = pd.read_csv("./赛题附件-输入输出格式说明/agv_trajectory.csv")  #
df = pd.read_csv("../2025 西门子 Xcelerator 公开赛 - 18831306395/agv_trajectory.csv")  #
df = df.dropna(axis=1, how='all')  # 删除空列

# 创建图形
fig, ax = plt.subplots(figsize=(14, 10))
ax.set_xlim(0, 22)
ax.set_ylim(0, 22)
ax.set_xlabel('X Coordinate')
ax.set_ylabel('Y Coordinate')
ax.set_title('AGV Movement Simulation with Map Points')
ax.grid(True, linestyle='--', alpha=0.6)

# 绘制起点（供件台）- 绿色三角形
for _, row in start_points.iterrows():
    ax.plot(row['x'], row['y'], 'g^', markersize=12)
    ax.text(row['x'] + 0.5, row['y'] + 0.3, row['name'],
            fontsize=9, bbox=dict(facecolor='white', alpha=0.7, edgecolor='none', boxstyle='round,pad=0.2'))

# 绘制终点（目的地）- 蓝色正方形
for _, row in end_points.iterrows():
    ax.plot(row['x'], row['y'], 'bs', markersize=10)
    ax.text(row['x'] + 0.5, row['y'] + 0.3, row['name'],
            fontsize=9, bbox=dict(facecolor='white', alpha=0.7, edgecolor='none', boxstyle='round,pad=0.2'))

# 添加地图标注
ax.text(0.5, 21, 'Factory Layout', fontsize=12, fontweight='bold', ha='center')
ax.text(1, 19, 'Start Points (Pick-up)', color='green', fontsize=10)
ax.text(1, 18, 'End Points (Delivery)', color='blue', fontsize=10)

# 创建存储AGV对象的字典
agv_artists = {}
agv_names = df['name'].unique()
colors = plt.cm.tab20(np.linspace(0, 1, len(agv_names)))

# 创建AGV的图形表示
for name, color in zip(agv_names, colors):
    # 主体圆形
    circle = Circle((0, 0), radius=0.4, fc=color, alpha=0.9, zorder=10)
    # 方向指示器
    wedge = Wedge((0, 0), 0.3, 0, 0, fc='black', alpha=0.9, zorder=11)
    # 名称标签
    text = ax.text(0, 0, name, fontsize=8, ha='center', va='center', zorder=12,
                   bbox=dict(facecolor='white', alpha=0.7, edgecolor='none', boxstyle='round,pad=0.1'))

    agv_artists[name] = {'circle': circle, 'wedge': wedge, 'text': text}
    ax.add_patch(circle)
    ax.add_patch(wedge)
    text.set_visible(True)

# 创建图例
legend_elements = [
    Line2D([0], [0], marker='^', color='w', markerfacecolor='green', markersize=10, label='Start Point (Pick-up)'),
    Line2D([0], [0], marker='s', color='w', markerfacecolor='blue', markersize=10, label='End Point (Delivery)'),
    Line2D([0], [0], marker='o', color='w', markerfacecolor='gray', markersize=10, label='AGV (No Load)'),
    Line2D([0], [0], marker='o', color='w', markerfacecolor='blue', markersize=10, label='AGV (Normal Load)'),
    Line2D([0], [0], marker='o', color='w', markerfacecolor='red', markersize=10, label='AGV (Emergency Load)'),
    Line2D([0], [0], marker='>', color='black', lw=0, markersize=10, label='AGV Direction')
]
ax.legend(handles=legend_elements, loc='upper right', fontsize=9)

# 时间戳文本
time_text = ax.text(0.02, 0.96, 'Timestamp: 0', transform=ax.transAxes, fontsize=12,
                    bbox=dict(facecolor='white', alpha=0.8, edgecolor='gray', boxstyle='round,pad=0.5'))

# 添加中文名称对照表
chinese_names = {
    "Optimus": "擎天柱",
    "Bumblebee": "大黄蜂",
    "Jazz": "爵士",
    "Sideswipe": "横炮",
    "Wheeljack": "千斤顶",
    "Ratchet": "救护车",
    "Ironhide": "铁皮",
    "Hound": "探长",
    "Smokescreen": "烟幕",
    "Megatron": "威震天",
    "Bluestreak": "飞毛腿",
    "RedAlert": "红色警报"
}

name_list = "\n".join([f"{eng}: {chinese_names[eng]}" for eng in agv_names])
ax.text(19.5, 21, 'AGV Name Translations:', fontsize=9, ha='right')
ax.text(19.5, 20, name_list, fontsize=8, ha='right', va='top',
        bbox=dict(facecolor='white', alpha=0.7, edgecolor='gray', boxstyle='round,pad=0.5'))


def init():
    """初始化动画"""
    time_text.set_text('Timestamp: 0')

    # 设置初始位置
    for name, agv in agv_artists.items():
        agv_data = df[(df['timestamp'] == 0) & (df['name'] == name)]
        if not agv_data.empty:
            x = agv_data['X'].values[0]
            y = agv_data['Y'].values[0]
            pitch = agv_data['pitch'].values[0]

            agv['circle'].center = (x, y)
            agv['text'].set_position((x, y))
            agv['wedge'].set_center((x, y))
            agv['wedge'].theta1 = pitch - 20
            agv['wedge'].theta2 = pitch + 20

            # 设置初始状态颜色
            loaded = agv_data['loaded'].values[0]
            emergency = agv_data['Emergency'].values[0]
            if loaded:
                if emergency:
                    agv['circle'].set_facecolor('red')
                else:
                    agv['circle'].set_facecolor('blue')
            else:
                agv['circle'].set_facecolor('gray')

    return [time_text] + [item for agv in agv_artists.values() for item in [agv['circle'], agv['wedge'], agv['text']]]


def update(frame):
    """更新每一帧"""
    current_time = frame
    time_text.set_text(f'Timestamp: {current_time}')

    # 获取当前时间戳的数据
    current_data = df[df['timestamp'] == current_time]

    for name, agv in agv_artists.items():
        agv_data = current_data[current_data['name'] == name]

        if not agv_data.empty:
            x = agv_data['X'].values[0]
            y = agv_data['Y'].values[0]
            pitch = agv_data['pitch'].values[0]
            loaded = agv_data['loaded'].values[0]
            emergency = agv_data['Emergency'].values[0]

            # 更新位置
            agv['circle'].center = (x, y)
            agv['text'].set_position((x, y))

            # 更新方向 (转换为matplotlib角度，0=东，90=北)
            agv['wedge'].set_center((x, y))
            agv['wedge'].theta1 = pitch - 20
            agv['wedge'].theta2 = pitch + 20

            # 更新状态颜色
            if loaded:
                if emergency:
                    agv['circle'].set_facecolor('red')
                else:
                    agv['circle'].set_facecolor('blue')
            else:
                agv['circle'].set_facecolor('gray')

    return [time_text] + [item for agv in agv_artists.values() for item in [agv['circle'], agv['wedge'], agv['text']]]


# 创建动画
timestamps = sorted(df['timestamp'].unique())
ani = animation.FuncAnimation(
    fig,
    update,
    frames=timestamps,
    init_func=init,
    blit=True,
    interval=800,  # 每800毫秒一帧
    repeat=False
)

plt.tight_layout()
plt.show()

# 保存动画（取消注释以保存）
# ani.save('./visualization/agv_simulation.mp4', writer='ffmpeg', fps=1, dpi=150)
```
## 评分代码

```python
import pandas as pd
import numpy as np
from sko.PSO import PSO
import json
import csv
import random
import heapq
import sys
import io
from collections import defaultdict
import os

class Score:
    # obstacles_map = [[0 for _ in range(20)] for _ in range(20)]

    task_file = os.path.join(os.getcwd(), "task_data.csv")
    agv_file = os.path.join(os.getcwd(), "agv_trajectory.csv")  # 使用本地路径
    map_file = os.path.join(os.getcwd(), "map_data.csv")
    resolved_file = os.path.join(os.getcwd(), "resolved.csv")
    score_file = os.path.join(os.path.dirname(__file__), "score.csv")

    agv_scrapped = []
    agvs = {}
    tasks = defaultdict(list)
    routes = []
    map_data = {}
    result = "0"

    strart_points = []
    loaded_agvs = {}
    original_points = {}
    end_points = {}
    end_point = {}

    agv_working = []
    urgent_tasks = []

    agv_work_num = 0
    max_timestamp = 0
    score = 0
    accident = 0
    finish_urgent_tasks = 0
    step = 1
    valid = True
    faild_route = {}
    error_routes = []

    faild_task = 0
    finish_task = 0
    timeout_task = 0
    error_routes_num = 0
    task_num = 0
    scrapped_route = []
    scrapped_agv = {}

    timeout_urgent = 0

    last_time = 0
    time_threshold = 300

    def __init__(self):
        with open(self.task_file, 'r') as fr:
            reader = csv.reader(fr)
            rows = list(reader)
            length = len(rows)
            for row in range(1, length):
                self.task_num += 1
                row = rows[row]
                order = row[0].split('-')
                task = {'task_id': row[0], 'start_point': row[1], 'end_point': row[2], 'priority': row[3],
                        'remaining_time': row[4], 'order': int(order[1])}
                if row[3] == 'Urgent' and row[0] not in self.urgent_tasks:
                    self.urgent_tasks.append(row[0])
                self.tasks[row[1]].append(task)
        print("task loaded")

        with open(self.map_file, 'r') as f:
            reader = csv.reader(f)
            rows = list(reader)
            length = len(rows)
            for row in range(1, length):
                row = rows[row]
                map_type = row[0]
                if map_type == 'start_point':
                    start_point = row[1]
                    pitch = row[4] if row[4] else 0
                    self.strart_points.append(start_point)
                    x = int(row[2])
                    y = int(row[3])
                    if x == 1:
                        x += 1
                    elif x == 20:
                        x -= 1
                    self.original_points[start_point] = {'x': x, 'y': y, 'pitch': pitch}
                elif map_type == 'end_point':
                    self.end_points[row[1]] = 0
                    self.end_point[row[1]] = {'x': int(row[2]), 'y': int(row[3]), 'pitch': pitch}
                elif map_type == 'agv':
                    pitch = row[4] if row[4] else 0
                    self.agvs[row[1]] = {'x': int(row[2]), 'y': int(row[3]), 'pitch': pitch, 'loaded': False,
                                         'Destination': ''}
                    self.agv_work_num += 1
        print("map loaded")

        with open(self.agv_file, 'r') as f:
            reader = csv.reader(f)
            rows = list(reader)
            length = len(rows)
            for row in range(1, length):
                route = {}
                row = rows[row]
                route['timestamp'] = int(row[0])
                route['name'] = row[1]
                route['pose'] = {'x': int(row[2]), 'y': int(row[3]), 'pitch': int(row[4])}
                route['loaded'] = row[5]
                route['Destination'] = row[6]
                route['Emergency'] = row[7]
                self.routes.append(route)
                self.max_timestamp = int(row[0])
        # 统计一共有多少次取货任务
        load_task_num = 0
        agv_status = {}
        # print(len(self.routes))

        print("agv loaded")

        with open(self.score_file, 'w', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(
                ['timestamp', 'Tiger', 'Dragon', 'Horse', 'Rabbit', 'Ox', 'Monkey', 'Beijing', 'Shanghai', 'Suzhou',
                 'Hangzhou', 'Nanjing', 'Wuhan', 'Changsha', 'Guangzhou', 'Chengdu', 'Xiamen', 'Kunming', 'Urumqi',
                 'Shenzhen', 'Dalian', 'Tianjin', 'Chongqing', 'score', 'AGVonWorking', 'accidentNum'])
        print("score csv created")

        with open(self.resolved_file, 'w', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(['timestamp', 'name', 'X', 'Y', 'pitch', 'loaded', 'destination', 'Emergency'])
        print("resolved csv created")
        # 生成障碍物地图，障碍物为：start_points、 end_points和 agvs的坐标
        # self.obstacles_c()

    # 判断a坐标是否在b坐标上下左右1格内,即判断a坐标是否在b坐标附近或相同

    def is_near(self, point, end_point):
        near_arr = [[end_point['x'] + 1, end_point['y']], [end_point['x'] - 1, end_point['y']],
                    [end_point['x'], end_point['y'] + 1], [end_point['x'], end_point['y'] - 1]]
        for near in near_arr:
            if point['x'] == near[0] and point['y'] == near[1]:
                return True
        return False

    # 生成障碍物地图，障碍物为：start_points、 end_points和 agvs的坐标
    def obstacles_c(self):
        start_points = self.map_data['start_points']
        end_points = self.map_data['end_points']
        for point in start_points:
            point = start_points[point]
            x = point[0] - 1
            y = point[1] - 1
            self.obstacles_map[y][x] = 2
        for point in end_points:
            point = end_points[point]
            x = point[0] - 1
            y = point[1] - 1
            self.obstacles_map[y][x] = 3
        for agv in self.map_data['agvs']:
            x = agv['pose'][0] - 1
            y = agv['pose'][1] - 1
            self.obstacles_map[y][x] = 1

        # for row in self.obstacles_map:
        #     print(' '.join(str(cell) for cell in row))

    # #判断agv是否在正确位置
    def is_agv_pos(self, route_pos, agv_pos):
        pitchs = [0, 90, 180, 270]
        flag = True
        if abs(route_pos['x'] - agv_pos['x']) <= 1 and abs(route_pos['y'] - agv_pos['y']) <= 1:
            flag = True
        else:
            flag = False

        if route_pos['pitch'] == agv_pos['pitch']:
            flag = True
        else:
            if route_pos['pitch'] in pitchs:
                flag = True
            else:
                flag = False

        return flag

    # 判断agv是否存在碰撞冲突
    def agv_collision(self, routes, timestamp):
        next_time = timestamp + 1
        current_coll = []
        next_coll = []
        # 获取当前时间戳和下一个时间戳的路径
        for route in routes:
            name = route['name']
            if name not in self.agv_scrapped:
                time = route['timestamp']
                if time == timestamp:
                    current_coll.append(route)
                elif time == next_time:
                    next_coll.append(route)

        def get_current_route(agv_name):
            for route in current_coll:
                if route['name'] == agv_name:
                    return route
            return None

        def get_next_route(agv_name):
            for route in next_coll:
                if route['name'] == agv_name:
                    return route
            return None

        for agv in current_coll:
            agv_pos = agv['pose']
            for next_agv in next_coll:
                if next_agv['name'] not in self.agv_scrapped:
                    next_agv_pos = next_agv['pose']

                    if agv_pos['x'] == next_agv_pos['x'] and agv_pos['y'] == next_agv_pos['y'] and agv['name'] != \
                            next_agv['name']:
                        agv_next_pos = get_next_route(agv['name'])
                        if agv_next_pos is not None:
                            agv_next_pos = agv_next_pos['pose']
                            agv_pitch = agv_pos['pitch']

                            next_agv_pitch = next_agv_pos['pitch']
                            if agv_pitch == next_agv_pitch:
                                continue
                            elif agv_pitch == 90 and next_agv_pitch != 270:
                                continue
                            elif agv_pitch == 270 and next_agv_pitch != 90:
                                continue
                            elif agv_pitch == 180 and next_agv_pitch != 0:
                                continue
                            elif agv_pitch == 0 and next_agv_pitch != 180:
                                continue

                            # if agv_pitch == 0:
                            #     agv_pitch = 180
                            # if next_agv_pos['pitch'] == 0:
                            #     next_agv_pitch = 180

                        else:
                            agv_next_pos = agv_pos
                            print(f'agv:{agv["name"]} 在第 {timestamp + 1} 秒时没有任何操作，默认为等待')
                            # print(f'agv:{agv["name"]} 在第 {timestamp + 1} 秒时没有任何操作，默认为等待')

                        print(
                            f'在第 {timestamp + 1} 秒时 agv:{agv["name"]} 和 agv:{next_agv["name"]} 发生碰撞， {agv["name"]}路径：{timestamp}{agv_pos}->{timestamp + 1}{agv_next_pos}  {next_agv["name"]}路径：{timestamp}{get_current_route(next_agv["name"])["pose"]}->{timestamp + 1}{next_agv_pos}')
                        print(
                            f'在第 {timestamp + 1} 秒时 agv:{agv["name"]} 和 agv:{next_agv["name"]} 发生碰撞， {agv["name"]}路径：{timestamp}{agv_pos}->{timestamp + 1}{agv_next_pos}  {next_agv["name"]}路径：{timestamp}{get_current_route(next_agv["name"])["pose"]}->{timestamp + 1}{next_agv_pos}')
                        self.accident += 1
                        self.scrapped_agv[agv["name"]] = timestamp + 1
                        self.scrapped_agv[next_agv['name']] = timestamp + 1

                        self.agv_working.remove(agv['name'])
                        self.agv_working.remove(next_agv['name'])
                        self.agv_scrapped.append(agv['name'])
                        self.agv_scrapped.append(next_agv['name'])
                        break

    # 判断agv是否重叠
    def agv_overlap(self, agvs, time):
        agv_pos_dict = {}
        for agv in agvs:

            timestamp = agv['timestamp']
            if agv['name'] not in self.agv_scrapped and time == timestamp:
                agv_pos = agv['pose']
                agv_pos_str = str(agv_pos['x']) + ',' + str(agv_pos['y'])
                if agv_pos_str not in agv_pos_dict:
                    agv_pos_dict[agv_pos_str] = agv['name']
                else:
                    self.agv_scrapped.append(agv['name'])
                    self.agv_scrapped.append(agv_pos_dict[agv_pos_str])
                    print(
                        f'在第{time}秒时 agv:{agv["name"]}{agv_pos} 和agv:{agv_pos_dict[agv_pos_str]}{agv_pos} 位置重叠')
                    print(
                        f'在第{time}秒时 agv:{agv["name"]}{agv_pos} 和agv:{agv_pos_dict[agv_pos_str]}{agv_pos} 位置重叠')
                    self.scrapped_agv[agv["name"]] = timestamp
                    self.scrapped_agv[agv_pos_dict[agv_pos_str]] = timestamp
                    # for agv_name in self.agv_scrapped:
                    #     if agv_name in self.agv_working:
                    #         self.agv_working.remove(agv_name)
                    self.accident += 1

    def score_tasks(self):
        # print(self.max_timestamp)
        for route in self.routes:
            timestamp = route['timestamp']

            name = route['name']

            # 如果agv已经损坏，则跳过该任务
            if name in self.agv_scrapped:
                self.scrapped_route.append(route)
                continue

            # 判断agv是否重叠
            self.agv_overlap(self.routes, timestamp)

            # 如果agv不在工作列表中，则添加到工作列表
            if name not in self.agv_working:
                self.agv_working.append(name)

            route_pos = route['pose']
            destination = route['Destination']
            # 判断agv位置是否正确
            if not self.is_agv_pos(route_pos, self.agvs[name]):
                print(f'{timestamp} agv {name} 未在正确位置')
                self.valid = False
                break

            # 判断agv是否存在对撞冲突
            if timestamp < self.max_timestamp - 1:
                self.agv_collision(self.routes, timestamp)

            # 判断agv是否按照正常路径行驶
            if not self.is_near(self.agvs[name], route['pose']):
                pos1 = self.agvs[name]
                pos2 = route['pose']
                if pos1['x'] != pos2['x'] and not pos1['y'] != pos2['y']:
                    self.scrapped_route.append(route)

                    if name not in self.error_routes:
                        print(
                            f'agv {name} 在第 {timestamp} 秒时未按照正常路径行驶, 当前路径为 {route["pose"]}，上一步路径为 {self.agvs[name]}')
                        print(
                            f'agv {name} 在第 {timestamp} 秒时未按照正常路径行驶, 当前路径为 {route["pose"]}，上一步路径为 {self.agvs[name]}')
                        self.error_routes.append(name)
                        self.error_routes_num += 1
                    continue
            #     # break

            # 判断该agv是否载货
            if route['loaded'] == 'true' and self.agvs[name]['loaded'] == 'false':
                # 判断取货点位置
                pos = []
                pos.append(route_pos['x'])
                pos.append(route_pos['y'])
                prev_pos = self.agvs[route['name']]
                if pos[0] != prev_pos['x'] or pos[1] != prev_pos['y']:
                    print(f'{timestamp} agv {route["name"]} 载货时存在坐标变化')
                elif route['pose']['pitch'] != prev_pos['pitch']:
                    print(f'{timestamp} agv {route["name"]} 载货时存在方向变化')

                for i in self.original_points:
                    original_point = []
                    original_point.append(self.original_points[i]['x'])
                    original_point.append(self.original_points[i]['y'])
                    load_flag = True
                    if original_point == pos:
                        for tmp in self.tasks[i]:
                            priority = tmp['priority']
                            task_id = tmp['task_id']
                            if tmp['end_point'] == destination:
                                if task_id == self.tasks[i][0]['task_id']:
                                    print(
                                        f'时间{timestamp} agv:{route["name"]}于{i}处载货, 目的地为{destination}, 取货任务{task_id}')
                                    # print(f'时间{timestamp} agv:{route["name"]}于{i}处载货, 目的地为{destination}, 取货任务{task_id}')
                                    self.loaded_agvs[route["name"]] = {'start': i, 'end': destination, 'task': tmp}
                                    # self.agv_working.append(route["name"])
                                    self.tasks[i].pop(0)
                                    load_flag = False

                                else:
                                    tmp_route = route.copy()
                                    tmp_route['loaded'] = 'false'
                                    tmp_route['Emergency'] = 'false'
                                    tmp_route['Destination'] = ''
                                    self.faild_route[route['name']] = tmp_route

                                    print(
                                        f'时间{timestamp} agv:{route["name"]}执行取货任务{task_id}, 正确任务应为：{self.tasks[i][0]["task_id"]}, 取货失败')
                                    self.faild_task += 1
                                    # print(f'时间{timestamp} agv:{route["name"]}执行取货任务{tmp}, 正确任务应为：{self.tasks[i][0]["task_id"]}, 取货失败')
                                    load_flag = False
                                break

                        if load_flag:
                            tmp_route = route.copy()
                            tmp_route['loaded'] = 'false'
                            tmp_route['Emergency'] = 'false'
                            tmp_route['Destination'] = ''
                            self.faild_task += 1
                            self.faild_route[route['name']] = tmp_route
                            print(
                                f'时间{timestamp} agv:{route["name"]}于{i}处载货, 目的地为{destination}, 取货失败， 无对应任务')
                            # print(f'时间{timestamp} agv:{route["name"]}于{i}处载货, 目的地为{destination}, 取货失败， 无匹配任务')

            elif route['loaded'] == 'false' and self.agvs[name]['loaded'] == 'true':
                if route['name'] in self.faild_route:
                    self.faild_route.pop(route['name'])
                if name in self.loaded_agvs:
                    if self.is_near(route['pose'], self.end_point[self.agvs[name]['Destination']]):
                        if route["name"] in self.error_routes:
                            self.error_routes.remove(route["name"])
                        else:
                            if timestamp <= self.time_threshold:
                                self.score += 1
                                self.last_time = timestamp
                                self.finish_task += 1
                            else:
                                self.timeout_task += 1
                            # print(f'时间{timestamp} agv:{route["name"]}于{self.agvs[name]['Destination']}卸货, 任务{self.loaded_agvs[name]["task"]["task_id"]}完成')

                            print(
                                f'时间{timestamp} agv:{route["name"]}于{self.agvs[name]["Destination"]}卸货, 任务{self.loaded_agvs[name]["task"]["task_id"]}完成')
                            # print(f'agv:{route["name"]} 完成')
                            task = self.loaded_agvs[name]['task']
                            self.end_points[self.agvs[name]['Destination']] += 1
                            if task['priority'] == 'Urgent':
                                remaining_time = int(task['remaining_time'])
                                if remaining_time > timestamp:
                                    self.score += 10
                                    if task['task_id'] in self.urgent_tasks:
                                        self.urgent_tasks.remove(task['task_id'])
                                        self.finish_urgent_tasks += 1
                                else:
                                    # print(f'时间{timestamp} agv:{route["name"]}于{self.agvs[name]["Destination"]}卸货, 任务{self.loaded_agvs[name]["task"]["task_id"]}超时, 规定完成时间为{task["remaining_time"]}秒， 扣5分。')
                                    print(
                                        f'时间{timestamp} agv:{route["name"]}于{self.agvs[name]["Destination"]}卸货, 任务{self.loaded_agvs[name]["task"]["task_id"]}超时, 规定完成时间为{task["remaining_time"]}秒， 扣5分。')
                                    self.urgent_tasks.remove(task['task_id'])
                                    self.timeout_urgent += 1
                                    self.score -= 5
                            self.loaded_agvs.pop(name)

            # elif route['name'] in self.faild_route:
            #     print('裁剪')
            #     continue

            self.agvs[name]['x'] = route_pos['x']
            self.agvs[name]['y'] = route_pos['y']
            self.agvs[name]['pitch'] = route_pos['pitch']
            self.agvs[name]['loaded'] = route['loaded']
            self.agvs[name]['Destination'] = route['Destination']
            # if route['timestamp'] > 11:
            #     break
            # if route['name'] in self.error_routes:
            #     continue
            if route['name'] in self.faild_route:
                tmp_route = self.faild_route[route['name']]
                tmp_route['pose'] = route['pose']
                tmp_route['timestamp'] = route['timestamp']

                self.append_resolved_csv(tmp_route)
            else:
                self.append_resolved_csv(route)

            # if destination in self.strart_points:
            # 从左至右的话分别是，时间戳，6列输入格口的待运输包裹数（agv接走就减少），16列输出格口的已运达的包裹数量（运到就+1），当前时刻得分，当前工作中AGV数量（有事故的就扣除对应数量agv），当前时刻累计事故数
            tmp = {}
            # writer.writerow(['timestamp', 'Tiger', 'Dragon', 'Horse', 'Rabbit', 'Ox', 'Monkey', 'Beijing', 'Shanghai', 'Suzhou', 'Hangzhou', 'Nanjing', 'Wuhan', 'Changsha', 'Guangzhou', 'Chengdu', 'Xiamen', 'Kunming', 'Urumqi', 'Shenzhen', 'Dalian', 'Tianjin', 'Chongqing', 'score', 'AGVonWorking', 'accidentNum'])
            if timestamp > self.step:
                self.append_score_csv()
                self.step = timestamp
                # print(self.end_points['Tianjin'])

        # print(self.agv_scrapped)
        # 应扣除分数为urgent_tasks*5
        self.score = self.score - len(self.urgent_tasks) * 5
        # 应扣除分数为accident * 10
        self.score = self.score - self.accident * 10
        self.append_score_csv()
        # if final_score < 0:
        #     final_score = 0

        agv_status = {}
        load_task_num = 0
        for route in self.routes:
            if route['name'] in agv_status:
                tmp_agv = {}
                tmp_agv = agv_status[route['name']]

                if agv_status and tmp_agv['name'] == route['name'] and tmp_agv['loaded'] == 'false' and route[
                    'loaded'] == 'true':
                    if route['name'] in self.scrapped_agv:
                        route_time = int(route['timestamp'])
                        scrapped_time = int(self.scrapped_agv[route['name']])
                        if route_time < scrapped_time:
                            # print(route, tmp_agv['name'])
                            load_task_num += 1
                    else:
                        # print(route, tmp_agv['name'])
                        load_task_num += 1

            load_agv = {}
            load_agv['name'] = route['name']
            load_agv['loaded'] = route['loaded']
            load_agv['loaded'] = route['loaded']
            load_agv['Destination'] = route['Destination']
            agv_status[route['name']] = load_agv
        # print(f'未取货任务： {self.tasks}')
        # logging.info(f'完成任务数：{self.score}, 高优先级快件送达数量:{self.finish_urgent_tasks} 加 {self.finish_urgent_tasks * 10} 分, 累计事故数：{self.accident}, 损坏车辆：{self.agv_scrapped}扣：{len(self.agv_scrapped) * 10} 分, 高优先级快件未送达数量：{len(self.urgent_tasks)} 扣：{len(self.urgent_tasks) * 5} 分, 最终得分：{final_score}')
        print('-' * 80, '最终得分', '-' * 80)
        print(
            f'总任务数：{self.task_num} 因车辆异常导致失败的任务数：{self.task_num - load_task_num} 异常取货任务数：{self.faild_task} 异常路径数：{self.error_routes_num} 超时任务：{self.timeout_task}  最终完成任务数：{self.finish_task}')
        print(
            f'高优先级快件送达数量:{self.finish_urgent_tasks} 加 {self.finish_urgent_tasks * 10} 分, 累计事故数：{self.accident}, 损坏车辆：{self.agv_scrapped} 扣：{self.accident * 10} 分, 高优先级快件未送达数量：{len(self.urgent_tasks)} 扣：{len(self.urgent_tasks) * 5} 分, 高优先级快件超时数量：{self.timeout_urgent} 扣：{self.timeout_urgent * 5} 分, 最终得分：{self.score}')
        print(f'最后一个有效任务完成时间：{self.last_time}')
        print(f'valid: {self.valid}')
        print(f'{self.score_file} 生成完成')
        print(f'{self.resolved_file} 生成完成')

    # 生成 resolved.csv 文件
    def append_resolved_csv(self, route):
        with open(self.resolved_file, 'a', newline='') as f:
            writer = csv.writer(f)
            pose = route['pose']
            writer.writerow([route['timestamp'], route['name'], pose['x'], pose['y'], pose['pitch'], route['loaded'],
                             route['Destination'], route['Emergency']])

    def append_score_csv(self):
        with open(self.score_file, 'a', newline='') as f:
            writer = csv.writer(f)
            writer.writerow([
                self.step,
                len(self.tasks[self.strart_points[0]]),
                len(self.tasks[self.strart_points[1]]),
                len(self.tasks[self.strart_points[2]]),
                len(self.tasks[self.strart_points[3]]),
                len(self.tasks[self.strart_points[4]]),
                len(self.tasks[self.strart_points[5]]),
                self.end_points['Beijing'],
                self.end_points['Shanghai'],
                self.end_points['Suzhou'],
                self.end_points['Hangzhou'],
                self.end_points['Nanjing'],
                self.end_points['Wuhan'],
                self.end_points['Changsha'],
                self.end_points['Guangzhou'],
                self.end_points['Chengdu'],
                self.end_points['Xiamen'],
                self.end_points['Kunming'],
                self.end_points['Urumqi'],
                self.end_points['Shenzhen'],
                self.end_points['Dalian'],
                self.end_points['Tianjin'],
                self.end_points['Chongqing'],
                self.score,
                self.agv_work_num - len(self.agv_scrapped),
                self.accident
            ])

```
# Baseline：引入PSO

```python
from typing import Any
from mcp.server.fastmcp import FastMCP
import pandas as pd
import numpy as np
from sko.PSO import PSO
import json
import csv
import random
import heapq
import sys
import io
from collections import defaultdict
import os
import types
import warnings
from abc import ABCMeta
from types import MethodType, FunctionType
from functools import lru_cache

# 设置随机数种子（确保结果可复现）
np.random.seed(6)

# 设置标准输出编码为UTF-8
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
# 定义转向成本为1秒
TURN_COST = 1
# 定义装卸货时间为1秒
LOAD_UNLOAD_TIME = 1
# 定义网格大小为21x21 (X:1-20, Y:1-20)
GRID_SIZE = (21, 21)

# 定义CSV文件路径
CSV_PATH = os.path.join(os.getcwd(), "agv_trajectory.csv")


def manhattan(p1, p2):
    """
    计算两点之间的曼哈顿距离
    
    Args:
        p1 (tuple): 第一个点的坐标 (x, y)
        p2 (tuple): 第二个点的坐标 (x, y)
        
    Returns:
        int: 两点之间的曼哈顿距离
    """
    # 计算两点之间的曼哈顿距离(|x1-x2| + |y1-y2|)
    return abs(p1[0] - p2[0]) + abs(p1[1] - p2[1])


def get_orientation(from_pos, to_pos):
    """
    计算从一个位置到另一个位置的方向向量对应的朝向角度
    
    Args:
        from_pos (tuple): 起始位置坐标 (x, y)
        to_pos (tuple): 目标位置坐标 (x, y)
        
    Returns:
        int/None: 朝向角度(0/90/180/270)或None(无方向变化)
    """
    # 计算从一个位置到另一个位置的方向向量
    dx, dy = to_pos[0] - from_pos[0], to_pos[1] - from_pos[1]
    # 根据方向向量确定朝向角度
    if dx > 0:
        return 0      # 向右
    elif dx < 0:
        return 180    # 向左
    elif dy > 0:
        return 90     # 向上
    elif dy < 0:
        return 270    # 向下
    else:
        return None   # 无方向变化


def a_star(start, goal, obstacles, grid_size=(21, 21)):
    """
    使用A*算法为AGV规划路径，AGV仅可以在X，Y方向移动
    
    Args:
        start (tuple): 起始位置坐标 (x, y)
        goal (tuple): 目标位置坐标 (x, y)
        obstacles (set): 障碍物位置集合
        grid_size (tuple): 网格大小 (默认为21x21)
        
    Returns:
        list: 从起始点到目标点的路径坐标列表，如果未找到路径则返回空列表
    """
    
    def neighbors(pos):
        """
        获取当前位置的所有有效邻居节点
        
        Args:
            pos (tuple): 当前位置坐标 (x, y)
            
        Yields:
            tuple: 有效的邻居节点坐标 (x, y)
        """
        # 定义获取邻居节点的内部函数
        x, y = pos  # 获取当前位置坐标
        # 遍历四个方向：右、左、上、下
        for dx, dy in [(1, 0), (-1, 0), (0, 1), (0, -1)]:
            nx, ny = x + dx, y + dy  # 计算邻居节点坐标
            # 检查邻居节点是否在有效范围内且不是障碍物
            if 1 <= nx <= grid_size[0] and 1 <= ny <= grid_size[1] and (nx, ny) not in obstacles:
                yield (nx, ny)  # 生成有效的邻居节点

    # 初始化优先队列，存储(优先级, 代价, 当前位置, 路径)
    frontier = []
    # 将起始节点加入优先队列，优先级=曼哈顿距离(启发式函数)，代价=0，路径=[起始点]
    heapq.heappush(frontier, (manhattan(start, goal), 0, start, [start]))
    visited = set()  # 创建已访问节点集合

    # 当优先队列不为空时继续搜索
    while frontier:
        # 从优先队列中取出优先级最低的节点
        _, cost, current, path = heapq.heappop(frontier)
        if current == goal:  # 如果当前节点是目标节点
            return path  # 返回找到的路径
        if current in visited:  # 如果当前节点已访问过
            continue  # 跳过该节点
        visited.add(current)  # 将当前节点标记为已访问
        # 遍历当前节点的所有有效邻居节点
        for neighbor in neighbors(current):
            if neighbor not in visited:  # 如果邻居节点未被访问过
                # 计算邻居节点的代价和优先级
                # 代价=当前代价+1(移动一步)，优先级=代价+到目标的曼哈坦距离
                heapq.heappush(frontier, (cost + 1 + manhattan(neighbor, goal), cost + 1, neighbor, path + [neighbor]))
    return []  # 如果未找到路径，返回空列表


def simulate_path_strict_load(name, path, start_time, initial_pitch, loaded, destination, emergency, taskid,
                              start_point, task_queues, t_q):
    """
    模拟AGV从当前位置前往取料点取料的路径过程，严格满足AGV运动规则
    
    Args:
        name (str): AGV名称
        path (list): 路径坐标列表
        start_time (int): 起始时间
        initial_pitch (int): 初始朝向
        loaded (bool): 是否已装载货物
        destination (str): 目的地
        emergency (bool): 是否为紧急任务
        taskid (str): 任务ID
        start_point (str): 起始点名称
        task_queues (dict): 任务队列
        t_q (int): 任务可执行时间
        
    Returns:
        tuple: (路径步骤列表, 最终时间, 最终朝向, 最终位置)
    """
    steps = []                      # 存储AGV每一步的状态
    t = start_time                  # 当前时间戳
    pitch = initial_pitch           # 当前朝向
    last = path[0]                  # 上一个位置，初始化为路径起点

    target_pickup_pos = path[-1]    # 取货点位置

    # 遍历路径中的每个位置（跳过起点）
    for current in path[1:]:
        new_pitch = get_orientation(last, current)  # 计算新的朝向

        # 若朝向变化，则先转向
        if new_pitch != pitch:
            steps.append({
                "timestamp": t,             # 转向发生的时间
                "name": name,               # AGV名称
                "X": last[0],               # X坐标保持不变
                "Y": last[1],               # Y坐标保持不变
                "pitch": new_pitch,         # 更新为新朝向
                "loaded": loaded,           # 是否载货
                "destination": destination if loaded else "",  # 目的地信息
                "Emergency": False,         # 紧急状态标志
                "taskid": taskid            # 任务ID
            })
            pitch = new_pitch               # 更新当前朝向
            t += 1                          # 时间增加1秒

        # 移动到下一个位置
        steps.append({
            "timestamp": t,                 # 移动发生的时间
            "name": name,                   # AGV名称
            "X": current[0],                # 新的X坐标
            "Y": current[1],                # 新的Y坐标
            "pitch": pitch,                 # 当前朝向
            "loaded": loaded,               # 是否载货
            "destination": destination if loaded else "",  # 目的地信息
            "Emergency": False,             # 紧急状态标志
            "taskid": taskid                # 任务ID
        })
        last = current                      # 更新上一个位置
        t += 1                              # 时间增加1秒

    # 到达取货点后，检查是否是当前可执行任务
    while True:
        # 获取当前取货点队列的第一个任务ID
        first_task_id = task_queues[start_point][0]["task_id"] if task_queues[start_point] and len(
            task_queues[start_point]) > 0 else None
        # 如果当前任务是队列首部任务且时间满足要求，则跳出循环
        if taskid == first_task_id and t >= t_q:
            break
        # 等待直到任务变为可执行
        steps.append({
            "timestamp": t,                 # 等待时间
            "name": name,                   # AGV名称
            "X": current[0],                # X坐标保持不变
            "Y": current[1],                # Y坐标保持不变
            "pitch": pitch,                 # 当前朝向
            "loaded": loaded,               # 是否载货
            "destination": destination if loaded else "",  # 目的地信息
            "Emergency": False,             # 紧急状态标志
            "taskid": taskid                # 任务ID
        })
        t += 1                              # 时间增加1秒

    # 完成取货操作
    steps.append({
        "timestamp": t,                     # 取货完成时间
        "name": name,                       # AGV名称
        "X": current[0],                    # X坐标
        "Y": current[1],                    # Y坐标
        "pitch": pitch,                     # 当前朝向
        "loaded": True,                     # 已载货
        "destination": destination,         # 目的地信息
        "Emergency": emergency,             # 紧急状态标志
        "task-id": taskid                   # 任务ID
    })

    return steps, t, pitch, last            # 返回路径步骤、最终时间、朝向和位置


def simulate_path_strict_unload(name, path, start_time, initial_pitch, loaded, destination, emergency, taskid):
    """
    模拟AGV从取料点到卸料点的路径过程，严格满足AGV运动规则
    
    Args:
        name (str): AGV名称
        path (list): 路径坐标列表
        start_time (int): 起始时间
        initial_pitch (int): 初始朝向
        loaded (bool): 是否已装载货物
        destination (str): 目的地
        emergency (bool): 是否为紧急任务
        taskid (str): 任务ID
        
    Returns:
        tuple: (路径步骤列表, 最终时间, 最终朝向, 最终位置)
    """
    steps = []                      # 存储AGV每一步的状态
    t = start_time                  # 当前时间戳
    pitch = initial_pitch           # 当前朝向
    last = path[0]                  # 上一个位置，初始化为路径起点

    # 遍历路径中的每个位置（跳过起点）
    for current in path[1:]:
        new_pitch = get_orientation(last, current)  # 计算新的朝向

        # 若朝向变化，则先转向
        if new_pitch != pitch:
            steps.append({
                "timestamp": t,             # 转向发生的时间
                "name": name,               # AGV名称
                "X": last[0],               # X坐标保持不变
                "Y": last[1],               # Y坐标保持不变
                "pitch": new_pitch,         # 更新为新朝向
                "loaded": loaded,           # 是否载货
                "destination": destination if loaded else "",  # 目的地信息
                "Emergency": emergency,     # 紧急状态标志
                "taskid": taskid            # 任务ID
            })
            pitch = new_pitch               # 更新当前朝向
            t += 1                          # 时间增加1秒

        # 移动到下一个位置
        steps.append({
            "timestamp": t,                 # 移动发生的时间
            "name": name,                   # AGV名称
            "X": current[0],                # 新的X坐标
            "Y": current[1],                # 新的Y坐标
            "pitch": pitch,                 # 当前朝向
            "loaded": loaded,               # 是否载货
            "destination": destination if loaded else "",  # 目的地信息
            "Emergency": emergency,         # 紧急状态标志
            "taskid": taskid                # 任务ID
        })
        last = current                      # 更新上一个位置
        t += 1                              # 时间增加1秒

    # 加入卸货动作
    steps.append({
        "timestamp": t,                     # 卸货时间
        "name": name,                       # AGV名称
        "X": current[0],                    # X坐标
        "Y": current[1],                    # Y坐标
        "pitch": pitch,                     # 当前朝向
        "loaded": "false",                  # 卸货后不再载货
        "destination": "",                  # 无目的地
        "Emergency": "false",               # 紧急状态结束
        "task-id": taskid                   # 任务ID
    })

    return steps, t, pitch, last            # 返回路径步骤、最终时间、朝向和位置


def reserve_simulated_steps(steps, reservation_table, agv_name):
    """
    将AGV的路径记录写入预约表，用于后续路径冲突判断
    
    Args:
        steps (list): AGV路径步骤列表
        reservation_table (dict): 预约表，键为(timestamp, (x, y))，值为AGV名称
        agv_name (str): AGV名称
    """
    # 遍历路径步骤，将每个时间点的位置信息记录到预约表中
    for step in steps:
        # 创建预约表的键，包含时间戳和位置坐标
        key = (step["timestamp"], (step["X"], step["Y"]))
        # 将当前AGV名称写入预约表对应位置
        reservation_table[key] = agv_name


def init_csv(agv_list):
    """
    初始化CSV_PATH的AGV轨迹文件，timestamp=0需要在最终轨迹文件中体现
    
    Args:
        agv_list (list): AGV列表，包含AGV的ID、位置和朝向信息
    """
    # 打开CSV文件进行写入
    with open(CSV_PATH, 'w', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        # 写入表头：时间戳、AGV名称、X坐标、Y坐标、朝向、是否载货、目的地、是否紧急、任务ID
        writer.writerow(["timestamp", "name", "X", "Y", "pitch", "loaded", "destination", "Emergency", "task-id"])
        # 为每个AGV写入初始状态（时间戳为0）
        for agv in agv_list:
            writer.writerow([
                0,                          # timestamp: 初始时间戳为0
                agv["id"],                  # name: AGV唯一标识
                agv["pose"][0],             # X: AGV初始X坐标
                agv["pose"][1],             # Y: AGV初始Y坐标
                agv["pitch"],               # pitch: AGV初始朝向角度
                "false",                    # loaded: 初始状态未装载货物
                "",                         # destination: 初始状态无目的地
                "false",                    # Emergency: 初始状态非紧急任务
                ""                          # task-id: 初始状态无任务ID
            ])


def append_to_csv(steps):
    """
    将路径步骤追加写入CSV文件
    
    Args:
        steps (list): AGV路径步骤列表
    """
    # 将路径步骤追加写入CSV文件
    with open(CSV_PATH, 'a', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        # 遍历每一步骤并写入CSV
        for step in steps:
            writer.writerow([
                step["timestamp"],              # 时间戳
                step["name"],                   # AGV名称
                step["X"],                      # X坐标
                step["Y"],                      # Y坐标
                step["pitch"],                  # 朝向角度
                str(step["loaded"]).lower(),    # 是否载货（转换为小写字符串）
                step["destination"],            # 目的地
                str(step["Emergency"]).lower(), # 是否紧急（转换为小写字符串）
                step.get("task-id", "")         # 任务ID（如果不存在则为空字符串）
            ])


def get_pickup_coord(start_point_name, original_coord):
    """
    根据起始点名称确定取料点坐标
    
    Args:
        start_point_name (str): 起始点名称
        original_coord (tuple): 原始坐标 (x, y)
        
    Returns:
        tuple: 取料点坐标 (x, y)
    """
    # 根据起始点名称确定取料点坐标
    if start_point_name in ["Tiger", "Dragon", "Horse"]:
        return (original_coord[0] + 1, original_coord[1])  # Tiger, Dragon, Horse取料点在X+1位置
    else:
        return (original_coord[0] - 1, original_coord[1])  # 其他取料点在X-1位置


def get_delivery_options(dest_coord):
    """
    获取卸料点周围的4个可选位置
    
    Args:
        dest_coord (tuple): 目标坐标 (x, y)
        
    Returns:
        list: 卸料点周围的4个可选位置坐标列表
    """
    # 获取卸料点周围的4个可选位置
    x, y = dest_coord
    return [(x + 1, y), (x - 1, y), (x, y + 1), (x, y - 1)]


def is_head_on_swap_conflict(path_steps, reservation_table):
    """
    检查路径中是否存在对穿（head-on swap）冲突
    
    Args:
        path_steps (list): 路径步骤列表
        reservation_table (dict): 预约表
        
    Returns:
        set: 冲突位置集合 {(x, y), ...}
    """
    conflict_points = set()
    # 检查路径中每一对连续步骤是否存在对穿冲突
    for i in range(len(path_steps) - 1):
        # 获取当前步骤和下一步骤的时间戳与位置信息
        now = (path_steps[i]["timestamp"], (path_steps[i]["X"], path_steps[i]["Y"]))
        nxt = (path_steps[i + 1]["timestamp"], (path_steps[i + 1]["X"], path_steps[i + 1]["Y"]))

        # 遍历预约表，检查是否存在对穿冲突
        for (t, pos), other_agv in reservation_table.items():
            # 如果当前时间点与预约时间点相同，且下一位置与预约位置相同
            if t == now[0] and pos == nxt[1]:
                # 计算回退位置的时间戳
                back_key = (t + 1, now[1])
                # 如果回退位置也在预约表中且为同一AGV，则存在对穿冲突
                if back_key in reservation_table and reservation_table[back_key] == other_agv:
                    # 记录对穿冲突的位置
                    conflict_points.add(now[1])
                    conflict_points.add(nxt[1])
    return conflict_points


def is_conflict(path_steps, reservation_table):
    """
    检测路径步骤中是否有与已有预约路径冲突的情况
    
    Args:
        path_steps (list): 路径步骤列表
        reservation_table (dict): 预约表
        
    Returns:
        tuple: (是否存在冲突(bool), 所有冲突位置的坐标集合(set))
    """
    conflict_points = set()

    # 检查静态冲突（同一时间同一位置）
    for step in path_steps:
        key = (step["timestamp"], (step["X"], step["Y"]))
        if key in reservation_table:
            conflict_points.add((step["X"], step["Y"]))

    # 检查对穿冲突
    swap_points = is_head_on_swap_conflict(path_steps, reservation_table)
    conflict_points.update(swap_points)

    return (len(conflict_points) > 0), conflict_points


def find_nearest_task_queue(agv_pos, task_queues, pickup_locks, start_points):
    """
    查找最近的未锁定任务队列
    
    Args:
        agv_pos (tuple): AGV当前位置 (x, y)
        task_queues (dict): 任务队列字典
        pickup_locks (dict): 取货点锁定状态字典
        start_points (dict): 起始点坐标字典
        
    Returns:
        str/None: 最近的未锁定取货点名称，如果没有则返回None
    """
    # 查找最近的未锁定任务队列
    candidates = []
    for sp in task_queues:
        # 如果该取货点有任务且未被锁定
        if task_queues[sp] and not pickup_locks[sp]:
            # 计算AGV到取货点的距离
            dist = manhattan(agv_pos, start_points[sp])
            candidates.append((dist, sp))
    if not candidates:
        return None
    # 返回距离最近的取货点
    return sorted(candidates, key=lambda x: x[0])[0][1]


# initialize MCP server
mcp = FastMCP("PathServer")


def calculatePath(agv_position: str, agv_task: str) -> str:
    """
    计算AGV路径的主要函数
    
    Args:
        agv_position (str): AGV位置数据文件路径
        agv_task (str): AGV任务数据文件路径
        
    Returns:
        str: 生成的CSV文件路径
    """
    def get_all_obstacles():
        """
        获取所有障碍物位置（起始点和终点位置）
        
        Returns:
            set: 所有障碍物位置集合
        """
        # 获取所有障碍物位置（起始点和终点位置）
        return set(start_points.values()) | set(end_points.values())

    # PSO任务分配函数
    def pso_task_assignment(free_agvs, available_tasks, agv_states, start_points):
        """使用PSO优化任务分配
        
        Args:
            free_agvs (list): 空闲AGV列表
            available_tasks (list): 可用任务列表
            agv_states (dict): AGV状态字典
            start_points (dict): 起始点坐标字典
            
        Returns:
            list: 最优任务分配结果
        """
        n_agvs = len(free_agvs)
        n_tasks = len(available_tasks)
        if n_agvs == 0 or n_tasks == 0:
            return []

        # 创建代价矩阵
        cost_matrix = np.zeros((n_tasks, n_agvs))
        for i, task in enumerate(available_tasks):
            for j, agv_id in enumerate(free_agvs):
                sp = task['start_point']
                start_pos = start_points[sp]
                agv_pos = agv_states[agv_id]['pos']
                d = manhattan(agv_pos, start_pos)
                if task['priority'].lower() == 'urgent':
                    d *= 0.5  # 紧急任务权重
                cost_matrix[i, j] = d

        # PSO适应度函数
        def fitness_func(assignment):
            """
            PSO适应度函数
            
            Args:
                assignment (list): 任务分配方案
                
            Returns:
                float: 适应度值（总成本）
            """
            total_cost = 0
            agv_count = np.zeros(n_agvs)
            for task_idx, agv_idx in enumerate(assignment):
                agv_idx = int(round(agv_idx))
                # 边界检查
                if agv_idx < 0 or agv_idx >= n_agvs:
                    return 1e9
                agv_count[agv_idx] += 1
                if agv_count[agv_idx] > 1:
                    return 1e9  # 惩罚重复分配
                total_cost += cost_matrix[task_idx, agv_idx]
            return total_cost

        # PSO参数设置
        dim = n_tasks
        lb = [0] * dim
        if n_agvs <= 1:
            ub = [1] * dim
        else:
            ub = [n_agvs - 1] * dim

        pso = PSO(func=fitness_func,
                  n_dim=dim,
                  pop=min(50, n_tasks * n_agvs),
                  max_iter=100,
                  lb=lb,
                  ub=ub,
                  w=0.8,
                  c1=0.5,
                  c2=0.5)

        pso.run()
        return np.round(pso.gbest_x).astype(int)

    # --- 数据加载 ---
    all_tasks = []
    agv_position = os.path.join(os.getcwd(), agv_position)
    agv_task = os.path.join(os.getcwd(), agv_task)

    # 读取任务数据
    with open(agv_task, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            all_tasks.append({
                "task_id": row["task_id"],
                "start_point": row["start_point"].strip(),
                "end_point": row["end_point"].strip(),
                "priority": row["priority"],
                "remaining_time": int(row["remaining_time"]) if row["remaining_time"] not in [None, "",
                                                                                              "None"] else None
            })

    # --- 地图数据 ---
    start_points, end_points, agv_list = {}, {}, []
    # 读取地图数据
    with open(agv_position, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            t, name = row["type"].strip(), row["name"].strip()
            x, y = int(row["x"]), int(row["y"])
            if t == "start_point":
                start_points[name] = (x, y)
            elif t == "end_point":
                end_points[name] = (x, y)
            elif t == "agv":
                agv_list.append({
                    "id": name,
                    "pose": (x, y),
                    "pitch": int(row["pitch"])
                })

    init_csv(agv_list)

    # ------------------ 初始化任务和AGV状态 ------------------
    task_queues = {}
    task_sequence = {}  # 记录每个取货点的任务顺序
    for task in all_tasks:
        sp = task["start_point"]
        if sp not in task_queues:
            task_queues[sp] = []
            task_sequence[sp] = []
        task_queues[sp].append(task)
        task_sequence[sp].append(task["task_id"])

    agv_queue = [agv["id"] for agv in agv_list]
    agv_states = {
        agv["id"]: {
            "pos": tuple(agv["pose"]),
            "pitch": agv["pitch"],
            "time": 1,
            "home": tuple(agv["pose"])
        } for agv in agv_list
    }

    retain_count = 12
    agv_queue = agv_queue[:retain_count]

    reservation_table = {}
    assigned_tasks = []

    pickup_locks = {sp: False for sp in task_queues}
    pickup_release_time = {sp: -1 for sp in task_queues}
    global_time = 1

    # 主循环：直到所有任务完成或达到时间限制
    while any(len(task_queues[sp]) > 0 for sp in task_queues) and global_time <= 300:
        # ✨✨ 解锁机制
        for sp in pickup_locks:
            if pickup_locks[sp] and global_time > pickup_release_time[sp]:
                pickup_locks[sp] = False

        agv_queue.sort(key=lambda aid: agv_states[aid]["time"])
        agv_progress = {aid: False for aid in agv_queue}

        # ================ PSO任务分配 ================
        # 收集空闲AGV和可用任务（只分配当前可执行的任务 - 队列首部任务）
        free_agvs = [agv for agv in agv_queue if agv_states[agv]['time'] <= global_time]
        available_tasks = []  # 只包含每个取货点的首部任务
        task_to_sp = {}

        # 新增逻辑：识别含有紧急任务的取货点，并确定紧急任务的位置
        urgent_start_points_info = {}  # 存储每个含有紧急任务的取货点及其紧急任务位置
        for sp in task_queues:
            queue = task_queues[sp]
            if queue and len(queue) > 0:
                for idx, task in enumerate(queue):
                    if task['priority'].lower() == 'urgent':
                        urgent_start_points_info[sp] = idx
                        break

        # 分离紧急任务和普通任务
        immediate_urgent_tasks = []  # 需要立即分配给最近AGV的任务
        pso_normal_tasks = []  # 通过PSO分配的普通任务
        immediate_urgent_task_to_sp = {}

        for sp in task_queues:
            queue = task_queues[sp]
            if queue and len(queue) > 0:
                first_task = queue[0]  # 只取第一个任务
                task_to_sp[first_task['task_id']] = sp

                # 如果该取货点含有紧急任务，则将该紧急任务及其之前的所有任务都视为紧急任务
                if sp in urgent_start_points_info:
                    i = 0
                    # 所有在紧急任务之前（包括紧急任务）的任务都视为紧急任务
                    urgent_index = urgent_start_points_info[sp]
                    # 但我们只处理队列头部的任务（即第一个任务）
                    if i <= urgent_index:  # 紧急任务就是第一个任务
                        immediate_urgent_tasks.append(first_task)
                        immediate_urgent_task_to_sp[first_task['task_id']] = sp
                        i += 1
                    else:
                        # 紧急任务不是第一个任务，第一个任务仍为普通任务
                        pso_normal_tasks.append(first_task)
                else:
                    # 普通取货点的第一个任务
                    pso_normal_tasks.append(first_task)

        # 优先处理需要立即分配的紧急任务
        if free_agvs and immediate_urgent_tasks:
            assigned_agv_set = set()

            for task in immediate_urgent_tasks:
                sp = immediate_urgent_task_to_sp[task['task_id']]
                ep = task["end_point"]
                priority = task["priority"]
                emergency = True
                taskid = task["task_id"]

                # 寻找最近的可用AGV
                nearest_agv = None
                min_distance = float('inf')
                start_pos = start_points[sp]

                for agv_id in free_agvs:
                    if agv_id in assigned_agv_set:
                        continue
                    agv_pos = agv_states[agv_id]['pos']
                    distance = manhattan(agv_pos, start_pos)
                    if distance < min_distance:
                        min_distance = distance
                        nearest_agv = agv_id

                if nearest_agv:
                    # 分配任务给最近的AGV
                    assigned_agv_set.add(nearest_agv)

                    # 执行任务分配逻辑（复制原有逻辑）
                    start_coord = get_pickup_coord(sp, start_points[sp])
                    end_coord_main = end_points[ep]
                    delivery_candidates = get_delivery_options(end_coord_main)

                    # 获取动态障碍
                    dynamic_obstacles = {
                        agv_states[other_agv]["pos"]
                        for other_agv in agv_queue
                        if other_agv != nearest_agv and agv_states[other_agv]["time"] <= global_time + 40
                    }

                    # 取料路径规划
                    path_to_pick = a_star(agv_states[nearest_agv]["pos"], start_coord,
                                          dynamic_obstacles | set(start_points.values()) | set(end_points.values()))
                    if path_to_pick:
                        # 模拟取料路径（修改后的函数）
                        steps1, t1, pitch1, pos1 = simulate_path_strict_load(
                            nearest_agv, path_to_pick, max(agv_states[nearest_agv]["time"], global_time),
                            agv_states[nearest_agv]["pitch"], False, ep, emergency, taskid, sp, task_queues,
                            pickup_release_time[sp]
                        )

                        # 冲突检测与解决
                        has_conflict, conflict_points = is_conflict(steps1, reservation_table)
                        if has_conflict:
                            path_to_pick = a_star(agv_states[nearest_agv]["pos"], start_coord,
                                                  conflict_points | dynamic_obstacles |
                                                  set(start_points.values()) | set(end_points.values()))
                            if path_to_pick:
                                steps1, t1, pitch1, pos1 = simulate_path_strict_load(
                                    nearest_agv, path_to_pick, max(agv_states[nearest_agv]["time"], global_time),
                                    agv_states[nearest_agv]["pitch"], False, ep, emergency, taskid, sp, task_queues,
                                    pickup_release_time[sp]
                                )
                                has_conflict, conflict_points = is_conflict(steps1, reservation_table)

                        if not has_conflict and path_to_pick:
                            t1 += 1

                            # 卸料路径规划
                            best_steps2, best_t2, best_pitch2, best_pos2 = None, float("inf"), None, None
                            for d in delivery_candidates:
                                path_to_deliver = a_star(pos1, d,
                                                         dynamic_obstacles | set(start_points.values()) | set(
                                                             end_points.values()))
                                if path_to_deliver:
                                    steps2, t2, pitch2, pos2 = simulate_path_strict_unload(
                                        nearest_agv, path_to_deliver, t1, pitch1, True, ep, emergency, taskid
                                    )

                                    # 冲突检测与解决
                                    has_conflict, conflict_points = is_conflict(steps2, reservation_table)
                                    if has_conflict:
                                        path_to_deliver = a_star(pos1, d,
                                                                 conflict_points | dynamic_obstacles |
                                                                 set(start_points.values()) | set(end_points.values()))
                                        if path_to_deliver:
                                            steps2, t2, pitch2, pos2 = simulate_path_strict_unload(
                                                nearest_agv, path_to_deliver, t1, pitch1, True, ep, emergency, taskid
                                            )
                                            has_conflict, conflict_points = is_conflict(steps2, reservation_table)

                                    if not has_conflict and path_to_deliver and best_t2 > t2:
                                        best_steps2, best_t2, best_pitch2, best_pos2 = steps2, t2, pitch2, pos2

                            if best_steps2:
                                best_t2 += 1
                                full_steps = steps1 + best_steps2
                                reserve_simulated_steps(full_steps, reservation_table, nearest_agv)
                                append_to_csv(full_steps)

                                assigned_tasks.append({
                                    "agv": nearest_agv,
                                    "start_point": sp,
                                    "end_point": ep,
                                    "priority": priority,
                                    "start_time": max(agv_states[nearest_agv]["time"], global_time),
                                    "agv_start_pose": agv_states[nearest_agv]["pos"],
                                    "agv_start_orientation": agv_states[nearest_agv]["pitch"]
                                })

                                agv_states[nearest_agv] = {
                                    "pos": best_pos2,
                                    "pitch": best_pitch2,
                                    "time": best_t2,
                                    "home": agv_states[nearest_agv]["home"]
                                }

                                # 设置锁与解锁时间（只有当前执行AGV才锁定取货点）
                                pickup_locks[sp] = True
                                pickup_release_time[sp] = t1
                                task_queues[sp].pop(0)  # 弹出已完成的任务
                                agv_progress[nearest_agv] = True

                                # 从free_agvs中移除已分配的AGV
                                if nearest_agv in free_agvs:
                                    free_agvs.remove(nearest_agv)

        # 剩余的普通任务使用PSO分配
        if free_agvs and pso_normal_tasks:
            best_assignment = pso_task_assignment(free_agvs, pso_normal_tasks, agv_states, start_points)
            assigned_agv_set = set()

            for i, agv_index in enumerate(best_assignment):
                if agv_index >= len(free_agvs):
                    continue
                agv_id = free_agvs[agv_index]
                if agv_id in assigned_agv_set:
                    continue
                assigned_agv_set.add(agv_id)

                task = pso_normal_tasks[i]
                sp = task_to_sp[task['task_id']]
                ep = task["end_point"]
                priority = task["priority"]
                emergency = True if priority.lower() == "urgent" else False
                taskid = task["task_id"]

                # 确保任务确实是该取货点的第一个任务
                if not task_queues[sp] or task_queues[sp][0]["task_id"] != taskid:
                    continue

                start_coord = get_pickup_coord(sp, start_points[sp])
                end_coord_main = end_points[ep]
                delivery_candidates = get_delivery_options(end_coord_main)

                # 获取动态障碍
                dynamic_obstacles = {
                    agv_states[other_agv]["pos"]
                    for other_agv in agv_queue
                    if other_agv != agv_id and agv_states[other_agv]["time"] <= global_time + 40
                }

                # 取料路径规划
                path_to_pick = a_star(agv_states[agv_id]["pos"], start_coord,
                                      dynamic_obstacles | set(start_points.values()) | set(end_points.values()))
                if not path_to_pick:
                    continue

                # 模拟取料路径（修改后的函数）
                steps1, t1, pitch1, pos1 = simulate_path_strict_load(
                    agv_id, path_to_pick, max(agv_states[agv_id]["time"], global_time),
                    agv_states[agv_id]["pitch"], False, ep, emergency, taskid, sp, task_queues, pickup_release_time[sp]
                )

                # 冲突检测与解决
                has_conflict, conflict_points = is_conflict(steps1, reservation_table)
                if has_conflict:
                    path_to_pick = a_star(agv_states[agv_id]["pos"], start_coord,
                                          conflict_points | dynamic_obstacles |
                                          set(start_points.values()) | set(end_points.values()))
                    if not path_to_pick:
                        continue
                    steps1, t1, pitch1, pos1 = simulate_path_strict_load(
                        agv_id, path_to_pick, max(agv_states[agv_id]["time"], global_time),
                        agv_states[agv_id]["pitch"], False, ep, emergency, taskid, sp, task_queues,
                        pickup_release_time[sp]
                    )
                    has_conflict, conflict_points = is_conflict(steps1, reservation_table)
                    if has_conflict:
                        continue
                t1 += 1

                # 卸料路径规划
                best_steps2, best_t2, best_pitch2, best_pos2 = None, float("inf"), None, None
                for d in delivery_candidates:
                    path_to_deliver = a_star(pos1, d,
                                             dynamic_obstacles | set(start_points.values()) | set(end_points.values()))
                    if not path_to_deliver:
                        continue
                    steps2, t2, pitch2, pos2 = simulate_path_strict_unload(
                        agv_id, path_to_deliver, t1, pitch1, True, ep, emergency, taskid
                    )

                    # 冲突检测与解决
                    has_conflict, conflict_points = is_conflict(steps2, reservation_table)
                    if has_conflict:
                        path_to_deliver = a_star(pos1, d,
                                                 conflict_points | dynamic_obstacles |
                                                 set(start_points.values()) | set(end_points.values()))
                        if not path_to_deliver:
                            continue
                        steps2, t2, pitch2, pos2 = simulate_path_strict_unload(
                            agv_id, path_to_deliver, t1, pitch1, True, ep, emergency, taskid
                        )
                        has_conflict, conflict_points = is_conflict(steps2, reservation_table)
                        if has_conflict:
                            continue
                    if best_t2 > t2:
                        best_steps2, best_t2, best_pitch2, best_pos2 = steps2, t2, pitch2, pos2

                if not best_steps2:
                    continue

                best_t2 += 1
                full_steps = steps1 + best_steps2
                reserve_simulated_steps(full_steps, reservation_table, agv_id)
                append_to_csv(full_steps)

                assigned_tasks.append({
                    "agv": agv_id,
                    "start_point": sp,
                    "end_point": ep,
                    "priority": priority,
                    "start_time": max(agv_states[agv_id]["time"], global_time),
                    "agv_start_pose": agv_states[agv_id]["pos"],
                    "agv_start_orientation": agv_states[agv_id]["pitch"]
                })

                agv_states[agv_id] = {
                    "pos": best_pos2,
                    "pitch": best_pitch2,
                    "time": best_t2,
                    "home": agv_states[agv_id]["home"]
                }

                # 设置锁与解锁时间（只有当前执行AGV才锁定取货点）
                pickup_locks[sp] = True
                pickup_release_time[sp] = t1
                task_queues[sp].pop(0)  # 弹出已完成的任务
                agv_progress[agv_id] = True
                # ================ PSO任务分配结束 ================

        # 当前时刻无任务，则补充轨迹
        if False in agv_progress.values():
            for agv in agv_queue:
                state = agv_states[agv]
                if state["time"] <= global_time and not agv_progress[agv]:
                    state["time"] += 1
                    idle_steps = []
                    idle_steps.append({
                        "timestamp": global_time,
                        "name": agv,
                        "X": state["pos"][0],
                        "Y": state["pos"][1],
                        "pitch": state["pitch"],
                        "loaded": "false",
                        "destination": "",
                        "Emergency": "false",
                        "task-id": ""
                    })
                    if not any((step["timestamp"], (step["X"], step["Y"])) in reservation_table for step in idle_steps):
                        reserve_simulated_steps(idle_steps, reservation_table, agv)
                        append_to_csv(idle_steps)

        # 全局时间推进
        if not any(agv_progress.values()):
            global_time += 1

    final_time = max(state["time"] for state in agv_states.values())

    finalize_trajectory_csv(agv_states, final_time)
    print(f"[INFO] 分配完成任务共计：{len(assigned_tasks)}，总时长：{final_time} 秒")
    return CSV_PATH


def fill_idle_steps(start_time, end_time, agv_state, agv_name):
    """补足AGV在空闲期间的轨迹，确保轨迹完整
    
    Args:
        start_time (int): 起始时间
        end_time (int): 结束时间
        agv_state (dict): AGV状态信息
        agv_name (str): AGV名称
    """
    append_to_csv([{
        "timestamp": start_time,                # 时间戳：记录AGV状态的时间点
        "name": agv_name,                       # AGV名称：标识当前AGV
        "X": agv_state["pos"][0],               # X坐标：AGV当前位置的X轴坐标
        "Y": agv_state["pos"][1],               # Y坐标：AGV当前位置的Y轴坐标
        "pitch": agv_state["pitch"],            # 朝向角度：AGV当前的朝向（0/90/180/270）
        "loaded": "false",                      # 是否载货：空闲状态下未载货
        "destination": "",                      # 目的地：空闲状态下无目的地
        "Emergency": "false",                   # 紧急状态：空闲状态下非紧急任务
        "task-id": ""                           # 任务ID：空闲状态下无任务ID
    }])


# 定义转换函数
def to_lower_str(x):
    """
    将输入转换为小写字符串
    
    Args:
        x (any): 输入值
        
    Returns:
        str: 小写字符串或原值（如果为NaN）
    """
    return str(x).lower() if pd.notna(x) else x


def finalize_trajectory_csv(agv_states, final_time, csv_path=CSV_PATH):
    """
    对所有 AGV 的轨迹进行时间补全并排序输出
    
    Args:
        agv_states (dict): AGV状态字典
        final_time (int): 最终时间
        csv_path (str): CSV文件路径
    """
    # 需要输出所有AGV从0时刻到最后一个任务结束时刻的所有AGV轨迹信息，即使AGV原地不动
    # 若在过程中未补充，需要在最终补全数据
    # 打开CSV文件以追加模式写入缺失的时间段数据
    with open(csv_path, 'a', newline='') as f:
        writer = csv.writer(f)
        # 遍历每个AGV的状态
        for agv_id, state in agv_states.items():
            last_time = state["time"]  # 获取该AGV最后记录的时间点
            # 从最后记录的时间点开始，补全到最终时间的所有时刻数据
            for t in range(last_time, final_time + 1):
                writer.writerow([
                    t,                          # 时间戳
                    agv_id,                     # AGV名称
                    state["pos"][0],            # X坐标
                    state["pos"][1],            # Y坐标
                    state["pitch"],             # 朝向角度
                    "false",                    # 是否载货（始终为false，因为空闲状态）
                    "",                         # 目的地（空闲时无目的地）
                    "false",                    # 是否紧急（始终为false）
                    ""                          # 任务ID（空闲时无任务）
                ])

    # 排序逻辑,最终输出的loaded、Emergency对应的列中"true","false"必须保持为小写，否则会影响评分
    # 读取CSV时直接转换
    # 读取CSV文件的列名，用于后续指定转换器
    col_names = pd.read_csv(csv_path, nrows=0).columns.tolist()
    # 使用converters确保第5列(loaded)和第7列(Emergency)转换为小写字符串
    df = pd.read_csv(csv_path, converters={col_names[5]: to_lower_str, col_names[7]: to_lower_str})
    # 按时间戳和AGV名称排序，确保轨迹顺序正确
    df.sort_values(by=["timestamp", "name"], inplace=True)
    # 将排序后的数据写回CSV文件
    df.to_csv(csv_path, index=False)
    print(f"[INFO] Final all AGV trajectory file with timestamp: saved to {csv_path}")

    # 基础碰撞类型检测方法
    # detect_collisions(csv_path='agv_trajectory.csv', output_path='agv_collisions.csv')


def detect_collisions(csv_path, output_path):
    """
    对最终输出的AGV轨迹CSV文件进行轨迹冲突判断
    
    Args:
        csv_path (str): AGV轨迹CSV文件路径
        output_path (str): 冲突记录输出文件路径
    """
    # 该示例参考方法
    # 提供对最终输出的AGV轨迹CSV文件进行轨迹冲突判断，该部分判断可能存在不完善的方面，仅供参考
    position_states = defaultdict(dict)  # {timestamp: {(x, y): agv_name}}
    agv_positions = defaultdict(dict)  # {agv: {timestamp: (x,y)}}

    # 读取CSV轨迹数据
    with open(csv_path, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            timestamp = int(row['timestamp'])          # 获取时间戳
            x, y = int(row['X']), int(row['Y'])        # 获取坐标
            agv = row['name']                          # 获取AGV名称
            position_states[timestamp][(x, y)] = agv   # 记录时间戳下坐标对应的AGV
            agv_positions[agv][timestamp] = (x, y)     # 记录AGV在时间戳下的坐标

    collisions = []

    # ---- 静态冲突 ----
    # 检查同一时间戳下，是否有多个AGV占据同一位置
    for timestamp in position_states:
        pos_counts = defaultdict(list)
        for pos, agv in position_states[timestamp].items():
            pos_counts[pos].append(agv)

        for pos, agvs in pos_counts.items():
            if len(agvs) > 1:
                collisions.append({
                    "timestamp": timestamp,
                    "X": pos[0],
                    "Y": pos[1],
                    "type": "static",
                    "AGVs": ", ".join(agvs)
                })

    # ---- 对穿冲突（去重）----
    # 检查两个AGV是否在相邻时间点交换了位置
    seen_crossings = set()
    for agv1 in agv_positions:
        for agv2 in agv_positions:
            if agv1 >= agv2:
                continue  # 避免重复和自身
            for t in agv_positions[agv1]:
                if (t + 1 not in agv_positions[agv1]) or (t + 1 not in agv_positions[agv2]):
                    continue

                p1_now = agv_positions[agv1][t]       # AGV1当前时刻位置
                p1_next = agv_positions[agv1][t + 1]  # AGV1下一时刻位置
                p2_now = agv_positions[agv2][t]       # AGV2当前时刻位置
                p2_next = agv_positions[agv2][t + 1]  # AGV2下一时刻位置

                # 判断是否发生对穿：AGV1移动到AGV2的位置，AGV2移动到AGV1的位置
                if p1_now == p2_next and p2_now == p1_next:
                    key = (t, tuple(sorted([agv1, agv2])))  # 生成唯一标识符避免重复记录
                    if key not in seen_crossings:
                        seen_crossings.add(key)
                        collisions.append({
                            "timestamp": t,
                            "X": p1_now[0],
                            "Y": p1_now[1],
                            "type": "crossing",
                            "AGVs": f"{agv1}, {agv2}"
                        })

    # ---- 保存检测结果 ----
    print(">>>>>>>>Collision record start：")
    with open(output_path, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=["timestamp", "X", "Y", "type", "AGVs"])
        writer.writeheader()
        writer.writerows(collisions)
        print(collisions)
    print(">>>>>>>>Collision record end.")

    print(f"[INFO] Detected {len(collisions)} collision events. Saved to {output_path}")


if __name__ == "__main__":
    # 运行路径规划
    print("开始路径规划...")
    calculatePath("map_data.csv", "task_data.csv")

    # 运行评分
    print("开始评分计算...")
    from local_score import Score

    score = Score()
    score.score_tasks()

    print("任务完成！")
```
## 输出结果

```python
开始路径规划...
[INFO] 分配完成任务共计：98，总时长：327 秒
开始评分计算...
task loaded
map loaded
agv loaded
score csv created
resolved csv created
时间8 agv:Ratchet于Rabbit处载货, 目的地为Kunming, 取货任务Rabbit-1
时间9 agv:Optimus于Tiger处载货, 目的地为Xiamen, 取货任务Tiger-1
时间9 agv:RedAlert于Monkey处载货, 目的地为Shanghai, 取货任务Monkey-1
时间10 agv:Ironhide于Horse处载货, 目的地为Chengdu, 取货任务Horse-1
时间17 agv:Bluestreak于Monkey处载货, 目的地为Chengdu, 取货任务Monkey-2
时间17 agv:Wheeljack于Ox处载货, 目的地为Suzhou, 取货任务Ox-1
时间19 agv:Hound于Dragon处载货, 目的地为Hangzhou, 取货任务Dragon-1
时间19 agv:Sideswipe于Rabbit处载货, 目的地为Hangzhou, 取货任务Rabbit-2
时间22 agv:Optimus于Xiamen卸货, 任务Tiger-1完成
时间23 agv:Ratchet于Kunming卸货, 任务Rabbit-1完成
时间23 agv:Smokescreen于Monkey处载货, 目的地为Beijing, 取货任务Monkey-3
时间25 agv:Bumblebee于Rabbit处载货, 目的地为Beijing, 取货任务Rabbit-3
时间26 agv:Jazz于Ox处载货, 目的地为Wuhan, 取货任务Ox-2
时间30 agv:RedAlert于Shanghai卸货, 任务Monkey-1完成
时间31 agv:Hound于Hangzhou卸货, 任务Dragon-1完成
时间31 agv:Ironhide于Chengdu卸货, 任务Horse-1完成
时间32 agv:Megatron于Tiger处载货, 目的地为Changsha, 取货任务Tiger-2
时间33 agv:Optimus于Rabbit处载货, 目的地为Chengdu, 取货任务Rabbit-4
时间34 agv:Wheeljack于Suzhou卸货, 任务Ox-1完成
时间38 agv:Bluestreak于Chengdu卸货, 任务Monkey-2完成
时间39 agv:Ratchet于Rabbit处载货, 目的地为Dalian, 取货任务Rabbit-5
时间40 agv:Jazz于Wuhan卸货, 任务Ox-2完成
时间40 agv:RedAlert于Tiger处载货, 目的地为Guangzhou, 取货任务Tiger-3
时间42 agv:Bumblebee于Beijing卸货, 任务Rabbit-3完成
时间43 agv:Hound于Dragon处载货, 目的地为Beijing, 取货任务Dragon-2
时间46 agv:Optimus于Chengdu卸货, 任务Rabbit-4完成
时间46 agv:Sideswipe于Hangzhou卸货, 任务Rabbit-2完成
时间47 agv:Megatron于Changsha卸货, 任务Tiger-2完成
时间47 agv:Ratchet于Dalian卸货, 任务Rabbit-5完成
时间49 agv:Ironhide于Tiger处载货, 目的地为Kunming, 取货任务Tiger-4
时间52 agv:Smokescreen于Beijing卸货, 任务Monkey-3完成
时间52 agv:Smokescreen于Beijing卸货, 任务Monkey-3超时, 规定完成时间为45秒， 扣5分。
时间55 agv:Wheeljack于Rabbit处载货, 目的地为Chengdu, 取货任务Rabbit-6
时间56 agv:Bluestreak于Tiger处载货, 目的地为Wuhan, 取货任务Tiger-5
时间56 agv:Hound于Beijing卸货, 任务Dragon-2完成
时间58 agv:Jazz于Horse处载货, 目的地为Shenzhen, 取货任务Horse-2
时间58 agv:Megatron于Dragon处载货, 目的地为Hangzhou, 取货任务Dragon-3
时间59 agv:RedAlert于Guangzhou卸货, 任务Tiger-3完成
时间63 agv:Optimus于Rabbit处载货, 目的地为Beijing, 取货任务Rabbit-7
时间66 agv:Bluestreak于Wuhan卸货, 任务Tiger-5完成
时间66 agv:Wheeljack于Chengdu卸货, 任务Rabbit-6完成
时间68 agv:Bumblebee于Ox处载货, 目的地为Beijing, 取货任务Ox-3
时间69 agv:Ironhide于Kunming卸货, 任务Tiger-4完成
时间71 agv:Smokescreen于Horse处载货, 目的地为Tianjin, 取货任务Horse-3
时间72 agv:Megatron于Hangzhou卸货, 任务Dragon-3完成
时间74 agv:Hound于Dragon处载货, 目的地为Chengdu, 取货任务Dragon-4
时间76 agv:Ratchet于Tiger处载货, 目的地为Tianjin, 取货任务Tiger-6
时间80 agv:Sideswipe于Rabbit处载货, 目的地为Xiamen, 取货任务Rabbit-8
时间82 agv:RedAlert于Tiger处载货, 目的地为Urumqi, 取货任务Tiger-7
时间83 agv:Bluestreak于Rabbit处载货, 目的地为Guangzhou, 取货任务Rabbit-9
时间85 agv:Jazz于Shenzhen卸货, 任务Horse-2完成
时间85 agv:Optimus于Beijing卸货, 任务Rabbit-7完成
时间85 agv:Wheeljack于Dragon处载货, 目的地为Dalian, 取货任务Dragon-5
时间88 agv:Smokescreen于Tianjin卸货, 任务Horse-3完成
时间89 agv:Bumblebee于Beijing卸货, 任务Ox-3完成
时间91 agv:Ironhide于Tiger处载货, 目的地为Beijing, 取货任务Tiger-8
时间91 agv:Sideswipe于Xiamen卸货, 任务Rabbit-8完成
时间95 agv:Optimus于Tiger处载货, 目的地为Shanghai, 取货任务Tiger-9
时间96 agv:Hound于Chengdu卸货, 任务Dragon-4完成
时间96 agv:Ratchet于Tianjin卸货, 任务Tiger-6完成
时间98 agv:Ironhide于Beijing卸货, 任务Tiger-8完成
时间100 agv:Bumblebee于Tiger处载货, 目的地为Tianjin, 取货任务Tiger-10
时间103 agv:Megatron于Rabbit处载货, 目的地为Hangzhou, 取货任务Rabbit-10
时间103 agv:Optimus于Shanghai卸货, 任务Tiger-9完成
时间103 agv:Wheeljack于Dalian卸货, 任务Dragon-5完成
时间104 agv:RedAlert于Urumqi卸货, 任务Tiger-7完成
时间105 agv:Smokescreen于Horse处载货, 目的地为Changsha, 取货任务Horse-4
时间107 agv:Bluestreak于Guangzhou卸货, 任务Rabbit-9完成
时间108 agv:Jazz于Dragon处载货, 目的地为Hangzhou, 取货任务Dragon-6
时间108 agv:Sideswipe于Tiger处载货, 目的地为Dalian, 取货任务Tiger-11
时间112 agv:Hound于Tiger处载货, 目的地为Chongqing, 取货任务Tiger-12
时间112 agv:Ironhide于Dragon处载货, 目的地为Shenzhen, 取货任务Dragon-7
时间116 agv:Smokescreen于Changsha卸货, 任务Horse-4完成
时间117 agv:Ratchet于Horse处载货, 目的地为Hangzhou, 取货任务Horse-5
时间119 agv:Jazz于Hangzhou卸货, 任务Dragon-6完成
时间122 agv:Bumblebee于Tianjin卸货, 任务Tiger-10完成
时间124 agv:Ratchet于Hangzhou卸货, 任务Horse-5完成
时间124 agv:Wheeljack于Tiger处载货, 目的地为Kunming, 取货任务Tiger-13
时间125 agv:RedAlert于Rabbit处载货, 目的地为Dalian, 取货任务Rabbit-11
时间125 agv:Sideswipe于Dalian卸货, 任务Tiger-11完成
时间127 agv:Smokescreen于Horse处载货, 目的地为Urumqi, 取货任务Horse-6
时间128 agv:Megatron于Hangzhou卸货, 任务Rabbit-10完成
时间130 agv:Optimus于Monkey处载货, 目的地为Nanjing, 取货任务Monkey-4
时间133 agv:RedAlert于Dalian卸货, 任务Rabbit-11完成
时间134 agv:Ironhide于Shenzhen卸货, 任务Dragon-7完成
时间134 agv:Jazz于Dragon处载货, 目的地为Chengdu, 取货任务Dragon-8
时间135 agv:Bluestreak于Rabbit处载货, 目的地为Shenzhen, 取货任务Rabbit-12
时间138 agv:Hound于Chongqing卸货, 任务Tiger-12完成
时间140 agv:Bumblebee于Dragon处载货, 目的地为Shenzhen, 取货任务Dragon-9
时间140 agv:Megatron于Horse处载货, 目的地为Changsha, 取货任务Horse-7
时间142 agv:Smokescreen于Urumqi卸货, 任务Horse-6完成
时间142 agv:Wheeljack于Kunming卸货, 任务Tiger-13完成
时间143 agv:Bluestreak于Shenzhen卸货, 任务Rabbit-12完成
时间150 agv:Megatron于Changsha卸货, 任务Horse-7完成
时间150 agv:Sideswipe于Dragon处载货, 目的地为Urumqi, 取货任务Dragon-10
时间151 agv:Ratchet于Rabbit处载货, 目的地为Chengdu, 取货任务Rabbit-13
时间152 agv:Hound于Ox处载货, 目的地为Shanghai, 取货任务Ox-4
时间152 agv:Ironhide于Monkey处载货, 目的地为Wuhan, 取货任务Monkey-5
时间152 agv:Jazz于Chengdu卸货, 任务Dragon-8完成
时间152 agv:Optimus于Nanjing卸货, 任务Monkey-4完成
时间152 agv:RedAlert于Dragon处载货, 目的地为Wuhan, 取货任务Dragon-11
时间156 agv:Wheeljack于Dragon处载货, 目的地为Changsha, 取货任务Dragon-12
时间159 agv:Bluestreak于Ox处载货, 目的地为Shanghai, 取货任务Ox-5
时间163 agv:Bumblebee于Shenzhen卸货, 任务Dragon-9完成
时间163 agv:Megatron于Horse处载货, 目的地为Chengdu, 取货任务Horse-8
时间163 agv:RedAlert于Wuhan卸货, 任务Dragon-11完成
时间163 agv:Smokescreen于Dragon处载货, 目的地为Guangzhou, 取货任务Dragon-13
时间165 agv:Ratchet于Chengdu卸货, 任务Rabbit-13完成
时间167 agv:Sideswipe于Urumqi卸货, 任务Dragon-10完成
时间167 agv:Wheeljack于Changsha卸货, 任务Dragon-12完成
时间169 agv:Hound于Shanghai卸货, 任务Ox-4完成
时间170 agv:Ironhide于Wuhan卸货, 任务Monkey-5完成
时间170 agv:Optimus于Dragon处载货, 目的地为Wuhan, 取货任务Dragon-14
时间173 agv:Jazz于Monkey处载货, 目的地为Shanghai, 取货任务Monkey-6
时间176 agv:Bluestreak于Shanghai卸货, 任务Ox-5完成
时间178 agv:Smokescreen于Guangzhou卸货, 任务Dragon-13完成
时间179 agv:RedAlert于Rabbit处载货, 目的地为Guangzhou, 取货任务Rabbit-14
时间183 agv:Optimus于Wuhan卸货, 任务Dragon-14完成
时间188 agv:Bumblebee于Dragon处载货, 目的地为Dalian, 取货任务Dragon-15
时间188 agv:Megatron于Chengdu卸货, 任务Horse-8完成
时间188 agv:Ratchet于Horse处载货, 目的地为Suzhou, 取货任务Horse-9
时间191 agv:Sideswipe于Rabbit处载货, 目的地为Urumqi, 取货任务Rabbit-15
时间194 agv:Ironhide于Horse处载货, 目的地为Chongqing, 取货任务Horse-10
时间196 agv:Jazz于Shanghai卸货, 任务Monkey-6完成
时间196 agv:Ratchet于Suzhou卸货, 任务Horse-9完成
时间197 agv:Smokescreen于Dragon处载货, 目的地为Dalian, 取货任务Dragon-16
时间198 agv:Hound于Ox处载货, 目的地为Changsha, 取货任务Ox-6
时间201 agv:Wheeljack于Rabbit处载货, 目的地为Dalian, 取货任务Rabbit-16
时间203 agv:RedAlert于Guangzhou卸货, 任务Rabbit-14完成
时间204 agv:Optimus于Horse处载货, 目的地为Suzhou, 取货任务Horse-11
时间206 agv:Bumblebee于Dalian卸货, 任务Dragon-15完成
时间209 agv:Jazz于Horse处载货, 目的地为Xiamen, 取货任务Horse-12
时间209 agv:Megatron于Dragon处载货, 目的地为Wuhan, 取货任务Dragon-17
时间209 agv:Wheeljack于Dalian卸货, 任务Rabbit-16完成
时间210 agv:Sideswipe于Urumqi卸货, 任务Rabbit-15完成
时间212 agv:Hound于Changsha卸货, 任务Ox-6完成
时间212 agv:Ironhide于Chongqing卸货, 任务Horse-10完成
时间212 agv:Optimus于Suzhou卸货, 任务Horse-11完成
时间213 agv:Bluestreak于Ox处载货, 目的地为Nanjing, 取货任务Ox-7
时间215 agv:Smokescreen于Dalian卸货, 任务Dragon-16完成
时间216 agv:Bumblebee于Rabbit处载货, 目的地为Suzhou, 取货任务Rabbit-17
时间218 agv:RedAlert于Horse处载货, 目的地为Xiamen, 取货任务Horse-13
时间219 agv:Ratchet于Monkey处载货, 目的地为Kunming, 取货任务Monkey-7
时间221 agv:Megatron于Wuhan卸货, 任务Dragon-17完成
时间221 agv:Wheeljack于Rabbit处载货, 目的地为Wuhan, 取货任务Rabbit-18
时间227 agv:Jazz于Xiamen卸货, 任务Horse-12完成
时间227 agv:Sideswipe于Horse处载货, 目的地为Chongqing, 取货任务Horse-14
时间228 agv:Hound于Ox处载货, 目的地为Suzhou, 取货任务Ox-8
时间230 agv:Ratchet于Kunming卸货, 任务Monkey-7完成
时间231 agv:Bluestreak于Nanjing卸货, 任务Ox-7完成
时间231 agv:Ironhide于Rabbit处载货, 目的地为Changsha, 取货任务Rabbit-19
时间235 agv:Wheeljack于Wuhan卸货, 任务Rabbit-18完成
时间237 agv:Bumblebee于Suzhou卸货, 任务Rabbit-17完成
时间237 agv:RedAlert于Xiamen卸货, 任务Horse-13完成
时间240 agv:Smokescreen于Horse处载货, 目的地为Hangzhou, 取货任务Horse-15
时间241 agv:Megatron于Ox处载货, 目的地为Guangzhou, 取货任务Ox-9
时间241 agv:Ratchet于Monkey处载货, 目的地为Kunming, 取货任务Monkey-8
时间245 agv:Hound于Suzhou卸货, 任务Ox-8完成
时间246 agv:Sideswipe于Chongqing卸货, 任务Horse-14完成
时间247 agv:Jazz于Horse处载货, 目的地为Urumqi, 取货任务Horse-16
时间247 agv:Smokescreen于Hangzhou卸货, 任务Horse-15完成
时间249 agv:Ironhide于Changsha卸货, 任务Rabbit-19完成
时间251 agv:Optimus于Rabbit处载货, 目的地为Suzhou, 取货任务Rabbit-20
时间252 agv:Ratchet于Kunming卸货, 任务Monkey-8完成
时间256 agv:RedAlert于Horse处载货, 目的地为Beijing, 取货任务Horse-17
时间256 agv:Wheeljack于Ox处载货, 目的地为Urumqi, 取货任务Ox-10
时间257 agv:Bluestreak于Monkey处载货, 目的地为Xiamen, 取货任务Monkey-9
时间260 agv:Jazz于Urumqi卸货, 任务Horse-16完成
时间261 agv:Bumblebee于Ox处载货, 目的地为Kunming, 取货任务Ox-11
时间261 agv:Megatron于Guangzhou卸货, 任务Ox-9完成
时间263 agv:Ironhide于Monkey处载货, 目的地为Hangzhou, 取货任务Monkey-10
时间269 agv:Hound于Ox处载货, 目的地为Urumqi, 取货任务Ox-12
时间272 agv:Bumblebee于Kunming卸货, 任务Ox-11完成
时间272 agv:RedAlert于Beijing卸货, 任务Horse-17完成
时间272 agv:Sideswipe于Horse处载货, 目的地为Xiamen, 取货任务Horse-18
时间273 agv:Wheeljack于Urumqi卸货, 任务Ox-10完成
时间274 agv:Bluestreak于Xiamen卸货, 任务Monkey-9完成
时间274 agv:Optimus于Suzhou卸货, 任务Rabbit-20完成
时间275 agv:Megatron于Horse处载货, 目的地为Hangzhou, 取货任务Horse-19
时间275 agv:Smokescreen于Monkey处载货, 目的地为Wuhan, 取货任务Monkey-11
时间279 agv:Ratchet于Horse处载货, 目的地为Tianjin, 取货任务Horse-20
时间280 agv:Ironhide于Hangzhou卸货, 任务Monkey-10完成
时间282 agv:Jazz于Ox处载货, 目的地为Urumqi, 取货任务Ox-13
时间283 agv:Megatron于Hangzhou卸货, 任务Horse-19完成
时间286 agv:Bumblebee于Monkey处载货, 目的地为Urumqi, 取货任务Monkey-12
时间286 agv:Hound于Urumqi卸货, 任务Ox-12完成
时间290 agv:Sideswipe于Xiamen卸货, 任务Horse-18完成
时间293 agv:Smokescreen于Wuhan卸货, 任务Monkey-11完成
时间296 agv:Ratchet于Tianjin卸货, 任务Horse-20完成
时间299 agv:Jazz于Urumqi卸货, 任务Ox-13完成
时间299 agv:RedAlert于Ox处载货, 目的地为Tianjin, 取货任务Ox-14
时间302 agv:Bumblebee于Urumqi卸货, 任务Monkey-12完成
时间304 agv:Bluestreak于Ox处载货, 目的地为Xiamen, 取货任务Ox-15
时间307 agv:RedAlert于Tianjin卸货, 任务Ox-14完成
时间309 agv:Ratchet于Ox处载货, 目的地为Shanghai, 取货任务Ox-16
时间315 agv:Bluestreak于Xiamen卸货, 任务Ox-15完成
时间326 agv:Ratchet于Shanghai卸货, 任务Ox-16完成
-------------------------------------------------------------------------------- 最终得分 --------------------------------------------------------------------------------
总任务数：100 因车辆异常导致失败的任务数：2 异常取货任务数：0 异常路径数：0 超时任务：4  最终完成任务数：94
高优先级快件送达数量:1 加 10 分, 累计事故数：0, 损坏车辆：[] 扣：0 分, 高优先级快件未送达数量：0 扣：0 分, 高优先级快件超时数量：1 扣：5 分, 最终得分：99
最后一个有效任务完成时间：299
valid: True
任务完成！
```

# 参考

 1. [https://www.bilibili.com/video/BV13x411G73y/?spm_id_from=333.337.search-card.all.click&vd_source=8125bd45b62d9eedeb53f0975906a586](https://www.bilibili.com/video/BV13x411G73y/?spm_id_from=333.337.search-card.all.click&vd_source=8125bd45b62d9eedeb53f0975906a586)
 2. [工艺魔方：Workflow Canvas](https://wfc.bd-iiot.com/projects)
 3. [比赛官网](https://www.siemens-x.com.cn/event-detail?eventId=700bf02d-da5c-46c6-ab18-374dc82438e6)

