---
title: 图深度学习、A_（A-Star）算法、EMD和VMD详解
date: 2026-01-04 08:04:00
tags:
  - "深度学习"
  - "图神经网络"
  - "A*算法"
  - "EMD"
  - "VMD"
categories:
  - "深度学习"
---

﻿@[TOC](图深度学习、A*（A-Star）算法、EMD和VMD详解)
## 模型推理（Model Inference）
指在机器学习或深度学习中，使用已训练好的模型对新输入的数据进行预测或分类的过程。简单来说，就是将训练好的模型应用于实际数据，输出结果。
## 图深度学习
图深度学习（Graph Deep Learning）是一种将深度学习技术应用于图数据结构的方法。图数据结构由节点（顶点）和边组成，能够表示复杂的关系和网络结构。图深度学习的主要目标是在这些结构化数据上执行任务，如节点分类、链接预测、图分类等。
### 图基础理论
图（Graph）是一种数学结构，用于表示对象之间的关系。在图中，对象被称为节点（Nodes 或 Vertices），而对象之间的关系则被称为边（Edges 或 Links）。图可以用来建模各种现实世界中的关系和网络，例如社交网络、交通网络、化学分子结构等。
#### 图的基本定义
**节点（Nodes 或 Vertices）**:

 - 图中的基本单元，表示实体或对象。
 - 通常用集合 ( V ) 表示，其中每个元素是一个节点。

**边（Edges 或 Links）**:

 - 连接两个节点的线段，表示节点之间的关系。
 - 通常用集合 ( E ) 表示，其中每个元素是一条边。
 - 边可以是有向的（Directed）或无向的（Undirected）：
 - 无向边: 不区分方向，表示双向关系。
 - 有向边: 区分方向，表示单向关系。

**邻接矩阵（Adjacency Matrix）**:

 - 一个 ( |V| \times |V| ) 的矩阵，用于表示图中节点之间的连接关系。
 - 如果节点 ( i ) 和节点 ( j ) 之间有一条边，则矩阵的第 ( i ) 行第 ( j ) 列的值为 1，否则为 0。

**邻接表（Adjacency List）**:

 - 一种链表结构，用于存储每个节点的邻居节点。
 - 每个节点对应一个链表，链表中的元素是该节点的所有邻居节点。

#### 图的类型
**无向图（Undirected Graph）**:

 - 边没有方向，表示双向关系。
 - 例如，社交网络中的好友关系。

**有向图（Directed Graph）**:

 - 边有方向，表示单向关系。

 - 例如，网页之间的链接关系。

**加权图（Weighted Graph）**:

 - 边有权重，表示关系的强度或距离。

 - 例如，交通网络中的道路长度。

**多重图（Multigraph）**:

 - 允许两个节点之间有多条边。
 - 例如，通信网络中的多条通信线路。

**混合图（Mixed Graph）**:

 - 同时包含有向边和无向边。

 - 例如，某些复杂的网络结构。

#### 图的基本术语
**度（Degree）**:

 - 一个节点的度是指与该节点相连的边的数量。
 - 在有向图中，分为入度（In-degree）和出度（Out-degree）。

**路径（Path）**:

 - 从一个节点到另一个节点的一系列边。
 - 路径的长度是指路径中边的数量。

**连通性（Connectivity）**:

 - 无向图中，如果任意两个节点之间都存在路径，则称该图为连通图。
 - 有向图中，如果任意两个节点之间都存在路径，则称该图为强连通图；如果任意两个节点之间存在一条路径（不考虑方向），则称该图为弱连通图。

**环（Cycle）**:

 - 一条从一个节点出发并最终返回该节点的路径。
 - 无环图（Acyclic Graph）是指不含环的图。

**子图（Subgraph）**:

 - 由原图的一部分节点和边组成的图。

**完全图（Complete Graph）**:

 - 每个节点都与其他所有节点相连的图。

### 图卷积神经网络

```python
import torch
import torch.nn.functional as F
from torch_geometric.nn import GCNConv
from torch_geometric.data import Data, DataLoader

# 假设我们有一个函数来加载和预处理数据
def load_and_preprocess_data():
    # 这里需要实现数据加载和预处理逻辑
    # 返回一个包含多个 Data 对象的列表
    return [Data(x=torch.randn(10, 5), edge_index=torch.tensor([[0, 1], [1, 0]]), edge_attr=torch.tensor([0.5]), y=torch.randn(1))]

# 加载数据
dataset = load_and_preprocess_data()
loader = DataLoader(dataset, batch_size=2, shuffle=True)

# 定义图神经网络模型
class GCN(torch.nn.Module):
    def __init__(self):
        super(GCN, self).__init__()
        self.conv1 = GCNConv(5, 16)
        self.conv2 = GCNConv(16, 1)

    def forward(self, data):
        x, edge_index, edge_weight = data.x, data.edge_index, data.edge_attr
        x = self.conv1(x, edge_index, edge_weight)
        x = F.relu(x)
        x = F.dropout(x, training=self.training)
        x = self.conv2(x, edge_index, edge_weight)
        return x.view(-1)

# 初始化模型、损失函数和优化器
model = GCN()
criterion = torch.nn.MSELoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.01)

# 训练模型
model.train()
for epoch in range(200):
    for data in loader:
        optimizer.zero_grad()
        out = model(data)
        loss = criterion(out, data.y)
        loss.backward()
        optimizer.step()
    print(f'Epoch {epoch+1}, Loss: {loss.item()}')

# 评估模型（这里仅作为示例，实际需要在验证集上进行评估）
model.eval()
for data in loader:
    with torch.no_grad():
        pred = model(data)
        print(f'Predicted: {pred.item()}, Actual: {data.y.item()}')

```
### Data
在使用 torch_geometric 库时，torch_geometric.data.Data 类是用于表示图数据的核心类。每个 Data 对象代表一个图，包含节点特征、边索引、边权重以及其他可能的图属性。下面是对 Data 类的详细解释：
#### 常用属性
**x**:

 - 类型: torch.Tensor
 - 描述: 节点特征矩阵。形状为 [num_nodes, num_node_features]，其中 num_nodes 是图中节点的数量，num_node_features 是每个节点的特征数量。

示例: x = torch.tensor([[1.0, 2.0], [3.0, 4.0]]) 表示两个节点，每个节点有两个特征。

**edge_index**:

 - 类型: torch.Tensor
 - 描述: 边索引矩阵。形状为 [2, num_edges]，其中 num_edges 是图中边的数量。edge_index 的每一列代表一条边的起点和终点节点索引。

示例: edge_index = torch.tensor([[0, 1], [1, 0]]) 表示节点 0 和节点 1 之间有一条双向边。

**edge_attr**:

 - 类型: torch.Tensor
 - 描述: 边特征矩阵。形状为 [num_edges, num_edge_features]，其中 num_edge_features 是每条边的特征数量。如果边没有特征，可以省略此属性。

示例: edge_attr = torch.tensor([[0.5], [0.5]]) 表示两条边的权重分别为 0.5。

**y**:

 - 类型: torch.Tensor
 - 描述: 目标值或标签。形状可以是 [num_nodes] 或 [1]，具体取决于任务。例如，在节点分类任务中，y 可以是每个节点的类别标签；在图回归任务中，y 可以是整个图的预测值。

示例: y = torch.tensor([1.0]) 表示图的目标值为 1.0。

**pos**:

 - 类型: torch.Tensor
 - 描述: 节点位置矩阵。形状为 [num_nodes, num_dimensions]，用于表示节点在空间中的位置。通常在图卷积神经网络中用于处理几何图。

示例: pos = torch.tensor([[0.0, 0.0], [1.0, 1.0]]) 表示两个节点的位置分别为 (0, 0) 和 (1, 1)。

**batch**:

 - 类型: torch.Tensor
 - 描述: 批处理索引。形状为 [num_nodes]，用于将多个图合并成一个批次。每个节点的值表示该节点属于哪个图。

示例: batch = torch.tensor([0, 0, 1, 1]) 表示前两个节点属于第一个图，后两个节点属于第二个图。
#### edge_index和torch.t() 
torch.t(input) 返回输入张量 input 的转置。输入张量必须是二维的（即形状为 [m, n] 的矩阵），转置后的形状为 [n, m]。

```python
原始张量:
tensor([[1., 2.],
        [2., 4.],
        [4., 8.],
        [6., 9.]])
使用 torch.t() 转置后的张量:
tensor([[1., 2., 4., 6.],
        [2., 4., 8., 9.]])
```

## A*（A-Star）算法
**启发式搜索算法**，用于在图形或网格中高效地找到从起点到目标点的**最优路径**。它结合了 **Dijkstra 算法**（保证找到最短路径）和**贪婪最佳优先搜索**（使用启发式函数提高效率）的优点。

### 核心思想

A* 算法通过评估每个可能的路径的“代价”来决定探索的方向。它为每个节点 `n` 计算一个评估函数 `f(n)`：

```
f(n) = g(n) + h(n)
```

- **`g(n)`：从起点到当前节点 `n` 的实际代价（已知代价）**。这是沿实际路径走过的距离、花费的时间等。
- **`h(n)`：从当前节点 `n` 到目标节点的估计代价（启发式代价）**。这是一个启发式函数，用于估计剩余路径的代价。
- **`f(n)`：通过节点 `n` 的路径的估计总代价**。算法优先探索 `f(n)` 最小的节点，因为它最有希望导向最优解。

## EMD (Empirical Mode Decomposition) 详解

EMD（经验模态分解） 是一种信号处理方法，用于将复杂信号分解为若干个本征模态函数（Intrinsic Mode Function, IMF）和一个残余项。EMD 方法特别适用于非线性和非平稳信号的分析。
### 主要步骤

 - 寻找极值点：找到信号中的所有极大值点和极小值点。
 - 构造上包络和下包络：通过插值方法（通常使用三次样条插值）连接所有极大值点和极小值点，分别得到上包络和下包络。
 - 计算均值包络：计算上包络和下包络的平均值。
 - 提取IMF：从原始信号中减去均值包络，得到一个新的信号。如果这个新的信号满足IMF的条件（即在任意时间区间内，极值点的数量和过零点的数量之差不超过1，并且在任意时间点上，局部极大值的平均值和局部极小值的平均值为零），则将其作为第一个IMF。否则，重复上述步骤直到满足条件。
 - 更新残余信号：从原始信号中减去已提取的IMF，得到新的残余信号。
 - 重复过程：对新的残余信号重复上述步骤，直到残余信号不再包含任何IMF成分或满足某种停止条件（如残余信号的频率低于某个阈值）。
## EMD-signal 详解
EMD-signal 是指使用EMD方法对信号进行分解后的结果。具体来说，EMD-signal 包含以下几个部分：
### IMF（Intrinsic Mode Function）：
定义：每个IMF是一个简单的振荡模式，具有明确的频率和幅度。
特点：IMF的频率从高到低排列，反映了信号的不同时间尺度特征。
应用：IMF可以用于进一步的信号分析，如频谱分析、时频分析等。
### 残余项（Residue）：
定义：经过多次分解后，剩余的无法再分解成IMF的部分。
特点：通常是一个趋势项或低频成分。
应用：残余项可以用于提取信号的长期趋势或低频特征。


```python
from PyEMD import EMD
import numpy as np
import matplotlib.pyplot as plt

# 生成一个示例信号
t = np.linspace(0, 1, 100)
s = np.sin(2 * np.pi * t) + np.sin(4 * np.pi * t) + np.random.normal(0, 0.1, t.shape)

# 创建EMD对象
emd = EMD()

# 进行EMD分解
IMFs = emd(s)

# 绘制结果
plt.figure(figsize=(10, 8))
plt.subplot(len(IMFs) + 1, 1, 1)
plt.plot(t, s, 'r')
plt.title("Original Signal")

for i, imf in enumerate(IMFs):
    plt.subplot(len(IMFs) + 1, 1, i + 2)
    plt.plot(t, imf, 'g')
    plt.title(f"IMF {i+1}")

plt.tight_layout()
plt.show()
```


## VMD（模态分解）详解
VMD (Variational Mode Decomposition) 是一种信号处理方法，用于将复杂信号分解为若干个模态分量。与传统的EMD（经验模态分解）相比，VMD具有更好的稳定性和鲁棒性。
### 主要特点
多模态分解：将原始信号分解为多个固有模态函数（IMF）。
变分问题：通过求解一个变分问题来实现信号的分解。
中心频率：每个模态分量都有一个中心频率，这使得VMD在频域上具有更好的分辨率。

```python
from vmdpy import VMD

alpha = 2000       # 模态带宽惩罚因子
tau = 0.0          # 时间延迟
K = 5              # 模态分量数量
DC = 0             # 是否包含直流分量
init = 1           # 初始化方式
tol = 1e-7         # 收敛容差

u, u_hat, omega = VMD(f, alpha, tau, K, DC, init, tol)
```
其中：
f 是输入信号。
u 是分解后的模态分量。
u_hat 是频域中的模态分量。
omega 是每个模态分量的中心频率。

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
for i in range(K):
    plt.subplot(K+1, 1, i+1)
    plt.plot(u[i], label=f'Mode {i+1}')
    plt.legend()
plt.subplot(K+1, 1, K+1)
plt.plot(f, label='Original Signal')
plt.legend()
plt.show()
```

参数说明
alpha：模态带宽惩罚因子，控制模态分量的平滑程度。
tau：时间延迟，通常设置为 0。
K：模态分量的数量。
DC：是否包含直流分量，0 表示不包含，1 表示包含。
init：初始化方式，1 表示基于傅里叶变换的初始化。
tol：收敛容差，用于判断算法何时停止迭代。
## 参考
1.[https://datawhalechina.github.io/grape-book/#/](https://datawhalechina.github.io/grape-book/#/)
