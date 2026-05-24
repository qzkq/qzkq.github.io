
---
title: 第三篇：NVIDIA cuOpt Python API 详解：LP、QP、MILP 建模与求解，附核心术语名词解释
date: 2026-05-24 08:00:00
tags:
  - "NVIDIA cuOpt"
  - "优化算法"
  - "LP"
  - "QP"
  - "MILP"
  - "GPU优化"
categories:
  - "运筹优化"
---


![在这里插入图片描述](/img/87292c1bf18b4dd2a29920054be6dee4.png)


在上一篇中，我们掌握了 OR-Tools 和 coptpy 两大求解器的实战技巧。本篇将聚焦于 NVIDIA cuOpt，一个 GPU 加速的优化库，详细讲解其 LP、QP、MILP 建模与求解的 Python API，并结合核心术语解释，帮助您在大规模优化问题中充分利用 GPU 的并行计算能力。

## 第一部分：cuOpt 概述与快速开始

### 1.1 什么是 NVIDIA cuOpt？

NVIDIA cuOpt™ 是一个 GPU 加速的优化库，专注于解决大规模优化问题。其核心能力包括：

- **线性规划 (LP)** —— 支持 PDLP、内点法、对偶单纯形等多种求解器
- **二次规划 (QP)** —— 支持凸二次规划
- **混合整数线性规划 (MILP)** —— 采用 GPU/CPU 混合求解策略
- **路径优化 (Routing)** —— 支持 TSP、VRP、PDP 等车辆路径问题

### 1.2 安装与验证

#### 使用 pip 安装（推荐）

```bash
# CUDA 12
pip install --extra-index-url=https://pypi.nvidia.com 'cuopt-cu12==26.2.*'
```

#### 使用 Docker（Windows 仅支持此方式）

```bash
docker pull nvidia/cuopt:latest-cuda12.9-py3.13
docker run --gpus all -it nvidia/cuopt:latest-cuda12.9-py3.13 /bin/bash  # --rm 
```

#### 验证安装

```python
import cudf
from cuopt import routing
print("cuOpt installed successfully")
```

---

## 第二部分：核心概念与 API 详解（含术语解释）

### 2.1 核心类：`Problem`

`Problem` 是建模的核心类，用于定义变量、约束、目标函数并执行求解。

```python
from cuopt.linear_programming.problem import Problem

problem = Problem("my_model")
```

#### 常用方法

| 方法/属性                               | 说明                                              |
| --------------------------------------- | ------------------------------------------------- |
| `addVariable(lb, ub, obj, vtype, name)` | 添加变量，返回 `Variable` 对象                    |
| `addConstraint(expr, name)`             | 添加约束，`expr` 可以是变量与系数的线性组合       |
| `setObjective(expr, sense)`             | 设置目标函数，`sense` 为 `MINIMIZE` 或 `MAXIMIZE` |
| `solve(settings)`                       | 求解模型                                          |
| `update()`                              | 在修改变量/约束后更新模型                         |
| `readMPS(file)` / `writeMPS(file)`      | 从/向 MPS 文件读写问题                            |
| `NumVariables` / `NumConstraints`       | 变量/约束数量（属性）                             |
| `Status`                                | 求解状态枚举                                      |
| `ObjValue`                              | 求解得到的目标函数值（属性）                      |

#### 求解状态 (`Status`)

| 状态            | 含义                     |
| --------------- | ------------------------ |
| `Optimal`       | 找到最优解               |
| `FeasibleFound` | 找到可行解（可能非最优） |
| `Infeasible`    | 问题无可行解             |
| `Unbounded`     | 目标函数无界             |
| `TimeLimit`     | 达到时间限制时停止       |

**用法示例**：

```python
if problem.Status.name == "Optimal":
    print("最优解")
elif problem.Status == cuopt.Status.FeasibleFound:
    print("可行解")
```

### 2.2 变量：`Variable`

变量通过 `addVariable` 创建：

```python
x = problem.addVariable(lb=0, ub=10, vtype=INTEGER, name="x")
x.setLowerBound(1)
x.setUpperBound(5)
x.setObjectiveCoefficient(3)
x.getValue()          # 求解后的值
x.getIndex()          # 变量索引
```

### 2.3 约束：`Constraint`

约束通过表达式与关系运算符定义：

```python
problem.addConstraint(x + 2*y <= 10, name="c1")
problem.addConstraint(x - y >= 0, name="c2")
problem.addConstraint(x + y == 5, name="c3")
```

约束对象可获取系数、对偶值、松弛量：

```python
c = problem.getConstraint("c1")
c.getCoefficient(x)   # 获取 x 的系数
c.DualValue           # 对偶值
c.Slack               # 松弛量
```

### 2.4 表达式

#### 线性表达式

支持变量与系数的线性组合：

```python
expr = 2*x + 3*y - z + 5
```

#### 二次表达式

支持两种构造方式：

**方式一：使用 `QuadraticExpression` 与矩阵**

```python
from cuopt.linear_programming.problem import QuadraticExpression

matrix = [[1.0, 0.0], [0.0, 2.0]]
vars = [x, y]
quad = QuadraticExpression(qmatrix=matrix, qvars=vars)
```

**方式二：直接通过变量相乘**

```python
quad_expr = x*x + 2*x*y + y*y
```

### 2.5 求解器设置：`SolverSettings`

用于配置求解器参数：

```python
from cuopt.linear_programming.solver_settings import SolverSettings

settings = SolverSettings()
settings.set_parameter("time_limit", 60)      # 秒
settings.set_optimality_tolerance(1e-6)
```

#### 可配置参数列表

`SolverSettings` 支持丰富的参数，用于控制求解器的行为。以下按类别列出常用参数及其说明。

##### 通用参数（LP 与 MILP 均适用）

| 参数名            | 类型        | 说明                                                 | 默认值            |
| ----------------- | ----------- | ---------------------------------------------------- | ----------------- |
| `time_limit`      | `int/float` | 求解时间限制（秒）                                   | 无限制            |
| `num_cpu_threads` | `int`       | CPU 线程数，`0` 表示自动检测                         | `0`               |
| `log_to_console`  | `bool`      | 是否将求解日志输出到控制台                           | `True`            |
| `log_file`        | `str`       | 日志文件路径，空字符串表示不写入                     | `""`              |
| `solution_file`   | `str`       | 最优解（可行解）的输出文件路径                       | `""`              |
| `presolve`        | `int`       | 预求解策略：`0` 禁用，`1` 使用 Papilo，`2` 使用 PSLP | MIP: `1`, LP: `2` |

##### MILP 专用参数

| 参数名                  | 类型    | 说明                                                         | 默认值  |
| ----------------------- | ------- | ------------------------------------------------------------ | ------- |
| `heuristics_only`       | `bool`  | 仅使用 GPU 启发式算法寻找可行解（不进行分支定界）            | `False` |
| `mip_scaling`           | `bool`  | 是否对 MIP 模型进行数值缩放                                  | `True`  |
| `mip_absolute_gap`      | `float` | 绝对 MIP 间隙，当 `|目标值 - 下界| ≤ 该值` 时停止            | `1e-10` |
| `mip_relative_gap`      | `float` | 相对 MIP 间隙，当 `|目标值 - 下界| / max(1, |目标值|) ≤ 该值` 时停止 | `1e-4`  |
| `integrality_tolerance` | `float` | 整数变量的容差，小于该值的分数部分视为整数                   | `1e-5`  |

##### LP 专用参数

| 参数名                      | 类型   | 说明                                                         | 默认值  |
| --------------------------- | ------ | ------------------------------------------------------------ | ------- |
| `method`                    | `int`  | 求解方法：`0` 并发执行所有可用方法，`1` PDLP，`2` 对偶单纯形 | `0`     |
| `pdlp_solver_mode`          | `int`  | PDLP 求解模式：`0` Stable1，`1` Stable2，`2` Methodical1，`3` Fast1 | `1`     |
| `iteration_limit`           | `int`  | PDLP 最大迭代次数                                            | 无限制  |
| `crossover`                 | `bool` | 是否将 PDLP 的原始-对偶解交叉为基本可行解（提高精度）        | `False` |
| `infeasibility_detection`   | `bool` | 是否启用 PDLP 的不可行性检测（增加开销但能更快识别无解）     | `False` |
| `strict_infeasibility`      | `bool` | 是否使用严格不可行性检测（更严格但更慢）                     | `False` |
| `save_best_primal_solution` | `bool` | 是否保存 PDLP 过程中找到的最佳原始解                         | `False` |
| `first_primal_feasible`     | `bool` | 是否在找到第一个原始可行解后立即停止                         | `False` |

##### 精度容差参数

| 参数名                      | 类型    | 说明                 | 默认值 |
| --------------------------- | ------- | -------------------- | ------ |
| `absolute_primal_tolerance` | `float` | 原始可行性的绝对容差 | `1e-4` |
| `relative_primal_tolerance` | `float` | 原始可行性的相对容差 | `1e-4` |
| `absolute_dual_tolerance`   | `float` | 对偶可行性的绝对容差 | `1e-4` |
| `relative_dual_tolerance`   | `float` | 对偶可行性的相对容差 | `1e-4` |
| `absolute_gap_tolerance`    | `float` | 对偶间隙的绝对容差   | `1e-4` |
| `relative_gap_tolerance`    | `float` | 对偶间隙的相对容差   | `1e-4` |

##### 参数设置示例

```python
from cuopt.linear_programming.solver_settings import SolverSettings

settings = SolverSettings()

# 通用设置
settings.set_parameter("time_limit", 300)          # 5 分钟
settings.set_parameter("num_cpu_threads", 8)       # 使用 8 个 CPU 线程
settings.set_parameter("log_file", "solve.log")

# MILP 特定设置
settings.set_parameter("mip_relative_gap", 1e-3)   # 相对间隙 0.1%
settings.set_parameter("heuristics_only", False)   # 允许分支定界

# LP 特定设置
settings.set_parameter("method", 1)                # 使用 PDLP
settings.set_parameter("pdlp_solver_mode", 3)      # Fast1 模式
settings.set_parameter("iteration_limit", 50000)

# 精度调整（提高数值稳定性）
settings.set_parameter("absolute_primal_tolerance", 1e-6)
settings.set_parameter("relative_dual_tolerance", 1e-6)
```

### 2.6 优化领域核心术语详解

#### 2.6.1 线性规划 (LP)

| 术语                    | 解释                                                         |
| ----------------------- | ------------------------------------------------------------ |
| **线性规划 (LP)**       | 目标函数和约束条件均为线性的优化问题。标准形式：`min c^T x` s.t. `Ax = b`, `x >= 0`。 |
| **可行域**              | 所有满足约束条件的点构成的集合。在 LP 中，可行域是多面体（凸集）。 |
| **顶点 / 基本可行解**   | 可行域的极点，对应于系数矩阵中线性无关列构成的基所确定的解。单纯形法通过遍历顶点寻找最优解。 |
| **单纯形法**            | 一种经典的 LP 算法，通过从一个顶点移动到相邻的改善顶点来找到最优解。在实践中非常高效，但理论上是指数时间复杂度。 |
| **对偶**                | 每个 LP 问题（原始问题）都对应一个对偶问题。原始问题的约束变成对偶问题的变量，反之亦然。强对偶定理指出，若原始问题有最优解，则对偶也有最优解且目标值相等。 |
| **内点法**              | 一种在可行域内部迭代的算法，通过牛顿法等逐步逼近最优解。对大规模问题通常比单纯形法更快。cuOpt 支持的内点法（Barrier）通过 GPU 加速求解线性方程组。 |
| **松弛**                | 将约束放松（例如去掉整数约束）得到的更容易求解的问题。LP 松弛是 MILP 常用的下界计算手段。 |
| **灵敏度分析**          | 研究模型参数（系数、右端项）变化对最优解和最优值的影响。     |
| **退化**                | 当存在多个线性相关的约束通过同一顶点时，出现退化现象，可能导致单纯形法循环或收敛变慢。 |
| **对偶变量 / 影子价格** | 对偶问题的最优解，表示约束右端项单位变化对目标值的影响。在 cuOpt 中可通过 `Constraint.DualValue` 获取。 |
| **基本解**              | 单纯形法中的基础解，对应一组基变量。                         |
| **人工变量**            | 用于处理等式约束或不等式约束的两阶段法引入的辅助变量，用于构造初始基可行解。 |

#### 2.6.2 二次规划 (QP)

| 术语               | 解释                                                         |
| ------------------ | ------------------------------------------------------------ |
| **二次规划 (QP)**  | 目标函数是二次函数（`1/2 x^T Q x + c^T x`），约束为线性。如果 Q 是半正定矩阵，则为凸 QP，可全局求解；否则为非凸 QP，求解困难。 |
| **凸二次规划**     | 目标函数的 Hessian 矩阵 Q 半正定，此时问题是凸优化问题，局部最优即为全局最优。 |
| **KKT 条件**       | Karush–Kuhn–Tucker 条件，非线性规划的最优性必要条件。对于凸 QP，也是充分条件。 |
| **积极集法**       | 一种求解 QP 的算法，通过猜测哪些约束在最优解处“活跃”（取等号），逐步调整集合，最终满足 KKT 条件。 |
| **内点法 (QP)**    | 推广了 LP 的内点法，通过求解一系列线性化系统来逼近最优解。   |
| **半定规划 (SDP)** | QP 的一种推广，变量为对称矩阵，约束涉及半正定条件。          |
| **矩阵分解**       | 求解 QP 时常用 LDLT 分解或 Cholesky 分解来处理二次项矩阵，cuOpt 利用 cuDSS 加速这些分解。 |

#### 2.6.3 混合整数线性规划 (MILP)

| 术语                        | 解释                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **混合整数线性规划 (MILP)** | 线性约束下，部分变量为整数。通常表示为 `min c^T x + d^T y` s.t. `Ax + By <= b`, `x` 连续，`y` 整数。 |
| **整数规划 (IP)**           | 所有变量均为整数。                                           |
| **分支定界**                | 求解 MILP 的经典算法。通过递归地分支（对整数变量取值进行划分）并计算每个子问题的下界（通过松弛）来剪枝，最终找到最优解或证明最优性。 |
| **分支切割**                | 在分支定界的基础上，动态加入割平面（切割不等式）以收紧松弛，加速收敛。 |
| **割平面**                  | 从当前松弛解推导出的有效不等式，能够割掉非整数解而不割掉任何整数可行解。 |
| **启发式**                  | 用于快速寻找高质量可行解的方法。cuOpt 的 MILP 求解器在 GPU 上运行多种启发式（如局部搜索、可行性泵）。 |
| **MIP Gap**                 | 当前最好可行解（上界）与对偶界（下界）之间的相对或绝对差距。当 gap 小于设定阈值时，可认为已找到足够好的解。 |
| **松弛**                    | 去掉整数约束后的 LP 问题，用于计算下界。                     |
| **原始启发式**              | 在求解过程中尝试构造可行解的方法，例如松弛解取整、基于约束的局部搜索等。 |
| **预处理 / 预求解**         | 在求解前简化问题，如删除冗余约束、固定变量、系数简化等。cuOpt 支持 Papilo 和 PSLP 预求解器。 |
| **对偶界**                  | 通过求解松弛或割平面获得的下界。对于最小化问题，所有可行解的目标值都大于等于对偶界。 |
| **整数可行解**              | 满足所有整数约束的解。                                       |
| **节点**                    | 分支定界树中的一个子问题。                                   |
| **热启动 (Warm Start)**     | 将之前求解得到的解作为初始解传递给求解器，有助于加速相似问题的求解。cuOpt 的 PDLP 支持热启动。 |

#### 2.6.4 cuOpt 特有术语

| 术语                                          | 解释                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| **PDLP (Primal-Dual hybrid gradient for LP)** | 一种一阶方法，专为大规模 LP 设计。通过 GPU 高效执行稀疏矩阵-向量乘法，实现快速迭代。不需要求解线性方程组，因此内存占用低，适合超大规模问题。cuOpt 的 PDLP 支持热启动和不可行性检测。 |
| **交叉 (Crossover)**                          | PDLP 等一阶方法得到的解通常不是顶点解（基本可行解）。开启交叉后，求解器会在最优解附近进行一次转换，得到一个更符合单纯形法定义的顶点解，便于后续敏感度分析或热启动。 |
| **热启动 (Warm Start)**                       | 将之前求解得到的解（如 PDLP 的迭代结果）作为初始点传递给求解器，可以加速相似问题的求解。cuOpt 的 PDLP 支持热启动，适合参数连续变化的问题序列。 |
| **GPU 启发式**                                | cuOpt 在 GPU 上并行执行的多种启发式算法，用于快速找到 MILP 的可行解（上界）。这些启发式包括取整、可行性泵、RINS 等，在分支定界之前或节点中运行，显著提升找可行解的速度。 |
| **混合求解 (Hybrid CPU/GPU)**                 | cuOpt 的 MILP 求解策略：GPU 负责并行启发式和部分 LP 求解（如 PDLP），CPU 负责分支定界主循环和对偶单纯形等。这种分工利用了 GPU 的吞吐量优势和 CPU 的控制逻辑优势。 |

---

## 第三部分：示例详解

### 3.1 线性规划 (LP)

**问题**：
```
Maximize: x + y
Subject to:
    x + y <= 10
    x - y >= 0
    x, y >= 0
```

**代码**：

```python
from cuopt.linear_programming.problem import Problem, MAXIMIZE, CONTINUOUS

problem = Problem("Simple LP")
x = problem.addVariable(lb=0, vtype=CONTINUOUS, name="x")
y = problem.addVariable(lb=0, vtype=CONTINUOUS, name="y")

problem.addConstraint(x + y <= 10)
problem.addConstraint(x - y >= 0)

problem.setObjective(x + y, sense=MAXIMIZE)
problem.solve()

print(f"x = {x.getValue()}, y = {y.getValue()}, obj = {problem.ObjValue}")
```

**输出**：
```
x = 10.0, y = 0.0, obj = 10.0
```

### 3.2 二次规划 (QP)

**问题**：
```
Minimize: x^2 + y^2
Subject to:
    x + y >= 1
    0.75*x + y <= 1
    x, y >= 0
```

**代码**：

```python
from cuopt.linear_programming.problem import Problem, MINIMIZE

problem = Problem("Simple QP")
x = problem.addVariable(lb=0)
y = problem.addVariable(lb=0)

problem.addConstraint(x + y >= 1)
problem.addConstraint(0.75*x + y <= 1)

problem.setObjective(x*x + y*y, sense=MINIMIZE)
problem.solve()

print(f"x = {x.getValue()}, y = {y.getValue()}, obj = {problem.ObjValue}")
```

**输出**：
```
x = 0.5, y = 0.5, obj = 0.5
```

### 3.3 混合整数线性规划 (MILP)

**问题**：
```
Maximize: 5*x + 3*y
Subject to:
    2*x + 4*y >= 230
    3*x + 2*y <= 190
    x, y integer
```

**代码**：

```python
from cuopt.linear_programming.problem import Problem, INTEGER, MAXIMIZE

problem = Problem("Simple MILP")
x = problem.addVariable(vtype=INTEGER, name="x")
y = problem.addVariable(lb=10, ub=50, vtype=INTEGER, name="y")

problem.addConstraint(2*x + 4*y >= 230)
problem.addConstraint(3*x + 2*y <= 190)

problem.setObjective(5*x + 3*y, sense=MAXIMIZE)
problem.solve()

print(f"x = {x.getValue()}, y = {y.getValue()}, obj = {problem.ObjValue}")
```

**输出**：
```
x = 36.0, y = 41.0, obj = 303.0
```

---

## 第四部分：高级功能

### 4.1 MIP 回调：获取中间解

```python
from cuopt.linear_programming.internals import GetSolutionCallback

class MyCallback(GetSolutionCallback):
    def get_solution(self, solution, cost, bound, user_data):
        print(f"当前可行解目标值: {cost[0]:.2f}")

settings.set_mip_callback(MyCallback(), user_data)
```

### 4.2 PDLP 热启动

适用于 LP，可加速相似问题的求解：

```python
warm_data = problem.getWarmstartData()
settings.set_pdlp_warm_start_data(warm_data)
new_problem.solve(settings)
```

---

## 第五部分：VSCode 调试配置（Dev Containers 环境）

在容器内开发时，推荐使用 Dev Containers 扩展，并正确配置调试。

### 5.1 确认环境

- 已安装 Docker、Dev Containers、Python 扩展
- 已通过 Dev Containers 连接到 cuOpt 容器
- 在容器内打开项目文件夹（如 `/workspace`）

### 5.2 创建 `launch.json`

在项目根目录创建 `.vscode/launch.json`：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: 当前文件",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "cwd": "${workspaceFolder}",
            "env": {
                "PYTHONPATH": "${workspaceFolder}"
            }
        }
    ]
}
```

### 5.3 启动调试

1. 设置断点
2. 选择配置（如 “Python: 当前文件”）
3. 按 `F5` 启动调试

### 5.4 常见问题

| 问题             | 解决方法                                                   |
| ---------------- | ---------------------------------------------------------- |
| 断点灰色         | 确认 Python 扩展安装在容器内，且文件路径正确               |
| 调试启动立即退出 | 检查 cuOpt 是否已安装（`pip list | grep cuopt`）           |
| 无法导入 cuOpt   | 在 `devcontainer.json` 中通过 `postCreateCommand` 预装依赖 |

---

## 第六部分：求解日志详解

当您调用 `problem.solve()` 后，cuOpt 会在控制台输出详细的求解日志。以下基于一个实际 MILP 问题的日志进行逐段解析，说明每个部分的含义。

### 6.1 求解启动与环境信息

```
求解开始: 2026-03-25 06:01:14
Setting parameter time_limit to 3.600000e+03
Setting parameter mip_absolute_gap to 2.590000e+02
cuOpt version: 26.2.0, git hash: f73da24d, host arch: x86_64, device archs: 70-real,75-real,80-real,86-real,90a-real,100f-real,120a-real,120
CPU: Intel(R) Core(TM) i7-14650HX, threads (physical/logical): 12/24, RAM: 9.07 GiB
CUDA 12.9, device: NVIDIA GeForce RTX 5060 Laptop GPU (ID 0), VRAM: 7.96 GiB
CUDA device UUID: 65ffffffb0fffffff9fffffff7-ffffffa27
```

- **参数设置**：日志首先输出用户通过 `SolverSettings` 设定的参数，如 `time_limit`（秒）、`mip_absolute_gap`（绝对 MIP 间隙）。
- **cuOpt 版本**：显示版本号和 Git 哈希，有助于复现或报告问题。
- **硬件信息**：CPU 物理/逻辑核心数、内存大小；CUDA 版本、GPU 型号、显存容量、设备 UUID。

### 6.2 问题规模与数值特征

```
Solving a problem with 34023 constraints, 19013 variables (19013 integers), and 101064 nonzeros
Problem scaling:
Objective coefficents range:          [1e+00, 5e+00]
Constraint matrix coefficients range: [1e+00, 1e+04]
Constraint rhs / bounds range:        [0e+00, 1e+04]
Variable bounds range:                [1e+00, 1e+04]
```

- **问题维度**：34023 个约束、19013 个变量（全部为整数）、101064 个非零元。
- **系数范围**：显示目标函数系数、约束矩阵系数、约束右端项、变量边界的数值范围。范围跨度大（如 1~1e4）可能影响数值稳定性，求解器会自动进行缩放处理。

### 6.3 预求解（Presolve）

```
Original problem: 34023 constraints, 19013 variables, 101064 nonzeros
Calling Papilo presolver (git hash 741a2b9c)
Presolve status: reduced the problem
Presolve removed: 23239 constraints, 12163 variables, 66274 nonzeros
Presolved problem: 10784 constraints, 6850 variables, 34790 nonzeros
Objective function is integral
Papilo presolve time: 0.39
Objective offset 181699.000000 scaling_factor 1.000000
Model fingerprint: 0x90ee62ba
Running presolve!
After cuOpt presolve: 10784 constraints, 6850 variables, objective offset 181699.000000.
cuOpt presolve time: 19.43
```

- **Papilo 预求解器**：一种 MILP 预求解技术，可删除冗余约束、固定变量、合并系数等。此处移除了约 68% 的约束和 64% 的变量，显著减小问题规模。
- **预求解耗时**：`Papilo presolve time: 0.39` 秒，`cuOpt presolve time: 19.43` 秒（包含其他预处理步骤）。
- **目标偏移**：`Objective offset` 是常数项，求解器会在最终目标值中加上该偏移。
- **模型指纹**：用于标识问题实例，有助于调试和缓存。

### 6.4 求解 LP 根松弛（Root Relaxation）

```
Solving LP root relaxation in concurrent mode
Skipping column scaling
Dual Simplex Phase 1
Dual feasible solution found.
Dual Simplex Phase 2
 Iter     Objective           Num Inf.  Sum Inf.     Perturb  Time
    0 -7.7228000000000000e+04      97 1.32680957e+08 0.00e+00 20.29
    1 -7.7228000000000000e+04     100 1.27856386e+08 0.00e+00 20.29
 1000 -2.7904000000000000e+04      95 9.33854416e+04 0.00e+00 20.31
Removed perturbation of 2.59e-06.


Root relaxation solution found in 1674 iterations and 0.08s by Dual Simplex
Root relaxation objective +1.76600000e+03
```

- **并发模式**：同时运行 PDLP、内点法、对偶单纯形，此处由 Dual Simplex 率先完成。
- **对偶单纯形迭代**：表格显示迭代次数、目标值、不可行数、总不可行量、扰动值、时间。最终在 1674 次迭代后得到根松弛目标值 **1766**。
- **根松弛目标**：这是 MILP 的下界（对于最小化问题），记为 `Bound`。

### 6.5 分支定界过程（Branch and Bound）

```
 | Explored | Unexplored |    Objective    |     Bound     | IntInf | Depth | Iter/Node |   Gap    |  Time  |
H                            +1.051210e+05    +1.766000e+03                                98.4%     20.41
...
           0            0    +7.458000e+04    +1.766000e+03      224      0   2.4e+03      97.7%     20.68
Gomory    cuts : 37
MIR       cuts : 0
Knapsack  cuts : 1
Strong CG cuts : 6
Cut pool size  : 182
Size with cuts : 10814 constraints, 17664 variables, 1952214848 nonzeros
Strong branching using 23 threads and 224 fractional variables
Strong branching completed in 0.47s
Exploring the B&B tree using 23 threads
```

- **表头解释**：
  - `Explored`：已处理（求解过 LP 松弛）的节点数。
  - `Unexplored`：待处理的节点数（分支定界树中尚未探索的节点）。
  - `Objective`：当前找到的最好可行解（上界，对于最小化问题）。
  - `Bound`：当前全局下界（所有未处理节点的最低 LP 松弛值）。
  - `IntInf`：当前节点中整数变量取非整数值的数量。
  - `Depth`：当前节点在搜索树中的深度。
  - `Iter/Node`：平均每个节点求解 LP 的单纯形迭代次数。
  - `Gap`：当前 MIP 间隙 = `(Objective - Bound) / max(1, |Objective|)`。
  - `Time`：累计求解时间（秒）。

- **行首字母含义**：
  - `H`：表示通过**启发式**找到了一个新的可行解（提升上界）。
  - 普通数字行：表示一个节点（通常是根节点或某个子节点）的 LP 松弛求解结果。

- **割平面**：在根节点处添加了 Gomory 割、Knapsack 割、Strong CG 割等，收紧松弛，提高下界。切割后约束数从 10784 增至 10814，变量数从 6850 增至 17664（因引入辅助变量）。

- **强分支**：在根节点使用强分支策略，评估哪些变量分支后效果最好。使用 23 个线程在 0.47 秒内完成。

### 6.6 分支定界中期日志

```
 | Explored | Unexplored |    Objective    |     Bound     | IntInf | Depth | Iter/Node |   Gap    |  Time  |
H                            +3.592100e+04    +1.766000e+03                                95.2%     21.97
...
        1001          735    +1.018700e+04    +1.766000e+03      204     11   5.6e+01      83.2%     27.40
...
       27442        17186    +2.207000e+03    +1.766000e+03      191     39   2.5e+01      22.3%    143.29
H                            +1.972000e+03    +1.766000e+03                                13.1%    143.57
Explored 27442 nodes in 147.38s.
Absolute Gap 2.580000e+02 Objective 1.9720000000000000e+03 Lower Bound 1.7139999999999709e+03
Optimal solution found within absolute MIP gap tolerance (2.6e+02)
```

- **节点处理**：随着搜索进行，`Explored` 递增，`Gap` 逐步下降（上界降低或下界提升）。最终探索了 27442 个节点，耗时 147.38 秒。
- **绝对间隙**：最终 `Objective = 1972`，`Lower Bound = 1766`，绝对差 = 258。由于设置的 `mip_absolute_gap` 为 259，满足停止条件，求解器终止并报告找到最优解（在容差范围内）。

### 6.7 求解总结

```
Solution objective: 1972.000000 , relative_mip_gap 0.130832 solution_bound 1766.000000 presolve_time 19.827045 total_solve_time 147.403601 max constraint violation 0.000000 max int violation 0.000000 max var bounds violation 0.000000 nodes 27442 simplex_iterations 692638
求解结束: 2026-03-25 06:03:42, 耗时: 148.18秒
✓ 找到可行解! 目标值: 1972.00
```

- **最终统计**：
  - `objective`：最优解目标值。
  - `relative_mip_gap`：相对间隙 = (1972 - 1766) / 1972 ≈ 13.1%。
  - `solution_bound`：最终下界。
  - `presolve_time`：预求解总时间（含 Papilo 和 cuOpt 预处理）。
  - `total_solve_time`：求解总时间（包括预求解、根松弛、分支定界等）。
  - `max constraint violation`：最大约束违反量，0 表示严格满足。
  - `max int violation`：最大整数变量违反（因数值容差产生），0 表示所有整数变量取整。
  - `nodes`：处理的总节点数。
  - `simplex_iterations`：所有节点 LP 求解的单纯形迭代总数。
- **求解结束时间**：显示实际耗时，与 `total_solve_time` 略有差异（可能包含日志输出等开销）。

通过解读日志，您可以：

- 确认问题规模、数值特征，判断是否需要调整模型或参数。
- 观察预求解效果，判断是否有明显简化。
- 监控分支定界进度，了解收敛速度。
- 评估求解质量（间隙、可行解质量）。
- 排查性能瓶颈（如预求解耗时、节点过多、割平面效果不佳等）。

结合日志信息，您可以更有效地调整求解器参数（如时间限制、间隙容忍度、算法选择等），以平衡求解速度与精度。

## 第七部分：与其他优化求解器的对比

本节对比 **COPT**（杉数科技）、**OR-Tools (CP-SAT)**、**Gurobi** 以及 **cuOpt** 在 **LP、QP、MILP、Routing、CP** 五个方面的支持情况。

### 7.1 各问题类型支持详情

#### 7.1.1 线性规划 (LP)

| 求解器       | 支持                | 算法                                                     | 特点                            |
| ------------ | ------------------- | -------------------------------------------------------- | ------------------------------- |
| **COPT**     | ✅ 原生支持          | 内点法、对偶单纯形、并发                                 | 性能接近 Gurobi，支持大规模 LP  |
| **OR-Tools** | ✅ 支持（通过 GLOP） | 单纯形法                                                 | 轻量级，适合小规模 LP，非主打   |
| **Gurobi**   | ✅ 原生支持          | 内点法、单纯形、并发、交叉                               | 行业标杆，稳定高效              |
| **cuOpt**    | ✅ 原生支持          | PDLP（GPU）、内点法（GPU 加速）、对偶单纯形（CPU）、并发 | GPU 加速显著，适合大规模稀疏 LP |

#### 7.1.2 二次规划 (QP) / 凸二次规划

| 求解器       | 支持               | 算法                  | 特点                                   |
| ------------ | ------------------ | --------------------- | -------------------------------------- |
| **COPT**     | ✅ 支持凸 QP        | 内点法、积极集法      | 支持凸二次规划，非凸暂不支持           |
| **OR-Tools** | ❌ 无原生 QP 支持   | —                     | 仅支持线性目标                         |
| **Gurobi**   | ✅ 支持凸 QP 和 QCP | 内点法、积极集法      | 支持二次约束 (QCP)、二次目标           |
| **cuOpt**    | ✅ 支持凸 QP        | 通过 GPU 加速的求解器 | 支持二次表达式，但文档中未详述算法细节 |

#### 7.1.3 混合整数线性规划 (MILP)

| 求解器       | 支持                                 | 核心算法                  | GPU 加速        | 特点                                    |
| ------------ | ------------------------------------ | ------------------------- | --------------- | --------------------------------------- |
| **COPT**     | ✅ 原生支持                           | 分支切割、启发式、预处理  | ❌               | 国产标杆，支持大规模 MILP               |
| **OR-Tools** | ✅ 支持（通过 CP-SAT 或原 MILP 接口） | 分支定界、CP 混合         | ❌               | 对整数问题友好，但性能不如商业求解器    |
| **Gurobi**   | ✅ 原生支持                           | 分支切割、启发式、并行    | ❌               | 行业最强 MILP 求解器                    |
| **cuOpt**    | ✅ 原生支持                           | GPU 启发式 + CPU 分支定界 | ✅（启发式阶段） | GPU 加速启发式找可行解，适合大规模 MILP |

#### 7.1.4 路径优化 (Routing)

| 求解器       | 支持             | 算法                       | GPU 加速 | 特点                                              |
| ------------ | ---------------- | -------------------------- | -------- | ------------------------------------------------- |
| **COPT**     | ❌ 无原生路由模块 | —                          | ❌        | 需用户自行建模为 MILP                             |
| **OR-Tools** | ✅ 原生支持       | LNS、启发式、CP 模型       | ❌        | TSP、VRP、VRPTW、PDP 等，易用性好                 |
| **Gurobi**   | ❌ 无原生路由模块 | —                          | ❌        | 需用户建模为 MILP                                 |
| **cuOpt**    | ✅ 原生支持       | 启发式、元启发式、GPU 并行 | ✅        | 专为大规模路由设计，支持 TSP、CVRP、VRPTW、PDP 等 |

#### 7.1.5 约束规划 (CP)

| 求解器       | 支持                 | 核心机制                | 特点                               |
| ------------ | -------------------- | ----------------------- | ---------------------------------- |
| **COPT**     | ❌ 无 CP 支持         | —                       | —                                  |
| **OR-Tools** | ✅ 原生支持（CP-SAT） | 约束传播、区间传播、LNS | 支持全局约束、调度、排班，易用性强 |
| **Gurobi**   | ❌ 无 CP 支持         | —                       | —                                  |
| **cuOpt**    | ❌ 无 CP 支持         | —                       | 但路由模块含部分约束传播思想       |

### 7.2 综合对比表

| 功能 / 求解器  | COPT     | OR-Tools (CP-SAT) | Gurobi   | cuOpt            |
| -------------- | -------- | ----------------- | -------- | ---------------- |
| **LP**         | ✅ 强     | ✅ 弱（GLOP）      | ✅ 强     | ✅ GPU 加速强     |
| **QP**         | ✅ 凸 QP  | ❌                 | ✅ 凸 QP  | ✅ 支持（细节少） |
| **MILP**       | ✅ 强     | ✅ 中              | ✅ 极强   | ✅ GPU 启发式加速 |
| **Routing**    | ❌ 需建模 | ✅ 强（易用）      | ❌ 需建模 | ✅ GPU 加速强     |
| **CP**         | ❌        | ✅ 极强            | ❌        | ❌                |
| **GPU 加速**   | ❌        | ❌                 | ❌        | ✅ 核心优势       |
| **易用性**     | 中       | 高                | 高       | 中               |
| **开源/商业**  | 商业     | 开源              | 商业     | 商业（NVAIE）    |
| **适用规模**   | 大       | 中                | 大       | 超大（GPU 并行） |
| **Python API** | ✅        | ✅                 | ✅        | ✅                |

### 7.3 选型建议

- **传统 LP / MILP / QP 问题**：首选 **Gurobi**（性能最强），其次 **COPT**（国产替代，性价比高）。
- **大规模 LP / MILP（千万级变量）**：可尝试 **cuOpt**，利用 GPU 加速 PDLP 和启发式。
- **路径优化（VRP / TSP / PDP）**：
  - 中小规模、快速原型：**OR-Tools**。
  - 超大规模、生产环境：**cuOpt**（GPU 加速）。
- **排班、调度、组合约束问题**：唯一选择 **OR-Tools (CP-SAT)**。
- **需同时覆盖 LP/MILP 和路由**：可组合使用 **Gurobi + OR-Tools** 或 **cuOpt**（如果路由和 LP 都需 GPU）。

---

## 第八部分：更多资源

- **官方文档**：[NVIDIA cuOpt User Guide](https://docs.nvidia.com/cuopt/user-guide/latest/introduction.html)
- **Python API 参考**：[Routing API](https://docs.nvidia.com/cuopt/user-guide/latest/cuopt-python/routing/routing-api.html) | [LP/QP/MILP API](https://docs.nvidia.com/cuopt/user-guide/latest/cuopt-python/lp-qp-milp/lp-qp-milp-api.html)
- **示例代码**：[cuOpt Examples on GitHub](https://github.com/NVIDIA/cuopt-examples)
- **第三方建模语言支持**：[AMPL, GAMS, PuLP, JuMP](https://docs.nvidia.com/cuopt/user-guide/latest/thirdparty_modeling_languages/index.html)
