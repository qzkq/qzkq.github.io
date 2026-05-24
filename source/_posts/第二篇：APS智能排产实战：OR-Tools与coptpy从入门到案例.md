
---
title: 第二篇：APS智能排产实战：OR-Tools与coptpy从入门到案例
date: 2026-05-24 08:00:00
tags:
  - "APS"
  - "智能排产"
  - "OR-Tools"
  - "COPT"
  - "优化算法"
categories:
  - "运筹优化"
  - "智能制造"
---


![封面图](/img/b89cdf0806874402859c32b96366c436.png)

在上一篇中，我们了解了 APS 的基本理论和求解器选型的原则。本篇将深入技术细节，详细介绍如何使用 Google OR-Tools 和杉数 coptpy 构建实际的排产优化模型。我们将从快速安装开始，逐步掌握两大求解器的核心 API、高级建模技巧，并通过一个完整的两阶段生产排程案例串联所有知识。

## 第四部分：Google OR-Tools 完全指南：从求解器选型到实战应用

### 🚀 快速开始

#### 1. 安装

```bash
pip install ortools
```

**注意**：如果遇到 `OSError: [WinError 127] 找不到指定的程序`，通常是缺少 Microsoft Visual C++ Redistributable。请安装 [最新版 VC++ 运行库](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)，并尝试将 ortools 导入语句放在代码最前面。

#### 2. 通用建模流程（四步法）

无论使用哪种求解器，基本建模流程相似：

1.  **创建求解器**：选择合适的求解器后端（如 GLOP、SCIP、CP-SAT）。
2.  **定义变量**：创建决策变量（连续、整数或布尔值）。
3.  **添加约束**：用 `add()` 方法添加问题的限制条件。
4.  **设置目标并求解**：定义最大化或最小化的目标函数，调用 `solve()`。

---

### 📦 核心模块详解

#### pywraplp：线性规划/混合整数规划模块

**支持的求解器后端**

| 求解器 | 类型               | 特点                                  |
| :----- | :----------------- | :------------------------------------ |
| GLOP   | 线性规划 (LP)      | OR-Tools 内置，免费，适用于纯线性规划 |
| SCIP   | 混合整数规划 (MIP) | 开源，支持整数变量，功能强大          |
| CBC    | 混合整数规划 (MIP) | 开源 COIN-OR 项目的一部分             |
| GUROBI | 商业求解器         | 高性能，需要许可证                    |
| CPLEX  | 商业求解器         | IBM 产品，业界领先                    |
| XPRESS | 商业求解器         | 高性能优化器                          |

**常用方法示例**

```python
from ortools.linear_solver import pywraplp

solver = pywraplp.Solver.CreateSolver('SCIP')  # 注意 CreateSolver 大写 C
if not solver:
    raise Exception('未找到指定的求解器')

x = solver.NumVar(0, solver.infinity(), 'x')   # 连续变量
y = solver.IntVar(0, 100, 'y')                 # 整数变量
z = solver.BoolVar('z')                        # 0-1 变量

solver.Add(2 * x + 3 * y <= 100)               # 约束
solver.Add(x >= 5 * z)
solver.Add(x + y == 50)

solver.Maximize(5 * x + 8 * y + 2 * z)         # 目标

status = solver.Solve()                        # 求解

if status == pywraplp.Solver.OPTIMAL:
    print(f'x = {x.solution_value()}')
    print(f'y = {y.solution_value()}')
    print(f'最优目标值 = {solver.Objective().Value()}')  # 或者 solver.objective_value
else:
    print('问题无最优解。')
```

**`pywraplp` 求解器参数设置**

`pywraplp` 提供了访问不同底层求解器（如 CBC、SCIP、GLOP 等）的接口，其参数主要通过以下方法设置：

*   **通用方法（常用）**：使用 `set_solver_specific_parameters_string` 方法。此方法允许以字符串形式直接传递底层求解器的原生参数。
*   **特定方法**：部分通用参数（如时间限制）有独立的设置函数。

**不同求解器的关键参数示例**

| 参数类别   | 适用求解器 | 参数设置示例                                                 | 说明                                                         |
| :--------- | :--------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 时间限制   | 所有求解器 | `solver.set_time_limit(10000)`（单位：毫秒）                 | 设置求解最大计算时间。                                       |
| 输出控制   | CBC/SCIP   | `solver.set_solver_specific_parameters_string("logLevel 1")` | `logLevel 0` 静默，`1` 常规输出，`2` 详细输出。              |
|            | GLOP       | `solver.enable_output()`                                     | GLOP 默认不输出，调用此函数开启基础日志。                    |
| 最优间隙   | CBC/SCIP   | `solver.set_solver_specific_parameters_string("allowableGap 1e-5")` | 当最优解与理论下界的相对间隙小于此值时，可提前停止。对求“足够好”的解很有用。 |
| 启发式策略 | CBC        | `solver.set_solver_specific_parameters_string("heuristics on maxNodes 100")` | 开启启发式搜索并限制节点数，以在整数规划中更快找到可行解。   |
| 切割生成   | CBC        | `solver.set_solver_specific_parameters_string("gomory on cuts on passC 5")` | 开启 Gomory 切割等，加强整数规划求解，但可能增加单次迭代时间。 |

---

#### CP-SAT：约束规划模块

CP-SAT 结合了约束规划（CP）和布尔可满足性问题（SAT），适用于具有复杂逻辑和整数约束的问题。它的核心是 Lazy Clause Generation，将整数变量编码为布尔变量，并通过 SAT 求解器进行高效搜索。CP-SAT 内置了线性规划传播器（基于对偶单纯形），并支持大量高级约束（如 `add_all_different`、`add_circuit`、`add_multiplication_equality` 等）。

**⚠️ 重要提示**：OR-Tools v9.0+ 已全面转向 **PEP8 蛇形命名法（snake_case）**。原有的大驼峰命名法（如 `NewIntVar`）虽仍可用但已被标记为 deprecated（弃用）。本文档示例均采用最新的 **snake_case** 标准。

**核心建模方法**

```python
from ortools.sat.python import cp_model

model = cp_model.CpModel()

x = model.NewIntVar(0, 10, 'x')
y = model.NewIntVar(0, 10, 'y')
z = model.NewIntVar(0, 10, 'z')
b = model.NewBoolVar('b')

model.Add(2 * x <= 11)
model.Add(x != 5)

# 修正蕴含
model.Add(x == 7).OnlyEnforceIf(b)   # 如果 b 为真，则 x = 7

model.AddAllDifferent([x, y, z])

mult = model.NewIntVar(0, 100, 'mult')
model.AddMultiplicationEquality(mult, [x, y])

model.Maximize(x + 5)

solver = cp_model.CpSolver()
solver.parameters.max_time_in_seconds = 30.0
solver.parameters.log_search_progress = True
status = solver.Solve(model)

if status in (cp_model.OPTIMAL, cp_model.FEASIBLE):
    print(f'x = {solver.Value(x)}')
    print(f'目标值 = {solver.ObjectiveValue()}')
else:
    print('未找到可行解。')
```

**CP-SAT 高级建模特性**

CP-SAT 提供了许多高级约束，可以简化生产排程等复杂问题的建模：

| 约束类型      | 方法                                                   | 说明                                                         |
| :------------ | :----------------------------------------------------- | :----------------------------------------------------------- |
| 电路/路径约束 | `add_circuit`                                          | 强制选中的边构成一个回路，用于建模旅行商、车辆路径等。       |
| 多重回路      | `add_multiple_circuit`                                 | 允许多个回路（如多辆车从同一个仓库出发）。                   |
| 区间变量      | `new_interval_var`, `new_optional_interval_var`        | 表示具有开始、结束、长度的时间区间，可与 `add_no_overlap` 等配合使用。 |
| 不重叠约束    | `add_no_overlap`                                       | 确保一组区间互不重叠（一维排程）。                           |
| 二维不重叠    | `add_no_overlap_2d`                                    | 用于矩形排样、场地规划等二维问题。                           |
| 累积资源约束  | `add_cumulative`                                       | 限制同时使用的资源总量（如电力、人力）。                     |
| 水库约束      | `add_reservoir_constraint`                             | 跟踪库存/水位变化，确保始终在上下限内。                      |
| 自动机约束    | `add_automaton`                                        | 有限状态机约束，用于建模状态转换。                           |
| 允许/禁止赋值 | `add_allowed_assignments`, `add_forbidden_assignments` | 直接指定变量的合法或非法组合。                               |

这些高级约束大大减少了手工编码的工作量，且 CP-SAT 内部有高效的传播器，通常比手动转换为线性约束更高效。

**CP-SAT 求解器参数**

CP-SAT 求解器参数主要通过 `solver.parameters` 进行设置。

**设置方法示例：**

```python
solver = cp_model.CpSolver()
solver.parameters.max_time_in_seconds = 600
solver.parameters.absolute_gap_limit = 0.01
solver.parameters.num_search_workers = 8
solver.parameters.log_search_progress = True
# 可以自定义日志回调（例如输出到文件）
solver.log_callback = print
```

**查看所有参数：**

```python
print(str(solver.parameters))
```

**主要参数分类说明**

| 参数类别   | 参数名                    | 类型  | 说明与典型取值                                               |
| :--------- | :------------------------ | :---- | :----------------------------------------------------------- |
| 终止条件   | `max_time_in_seconds`     | float | 最大求解时间（秒）。超时后停止，返回当前最优解。例如：`7200`。 |
|            | `max_number_of_conflicts` | int   | 最大冲突次数限制。冲突指导致回溯的赋值矛盾，用于控制搜索深度。 |
|            | `absolute_gap_limit`      | float | 绝对最优间隙。当 `当前解 - 最优下界 ≤ 此值` 时停止。         |
|            | `relative_gap_limit`      | float | 相对最优间隙。当 `(当前解 - 最优下界) / 最优下界 ≤ 此值` 时停止。 |
| 并行求解   | `num_search_workers`      | int   | 并行工作线程数。CP-SAT 采用 portfolio 并行策略，每个 worker 运行不同的子求解器。通常设为 CPU 核心数。例如：`8`。 |
| 预处理     | `cp_model_presolve`       | bool  | 启用预处理。默认为 `True`，简化模型，通常能加速求解。        |
|            | `cp_model_probing_level`  | int   | 探测级别（0-2）。值越高，预处理时推理越强，但耗时可能增加。  |
| 线性化级别 | `linearization_level`     | int   | 线性化级别（0-2）。值越高，尝试将更多约束线性化，可能增强 LP 传播，但也增加模型大小。 |
| 启发式策略 | `use_objective_lb_search` | bool  | 基于目标下界的搜索。为 `True` 时，搜索更关注提升目标下界。   |
|            | `use_lns_only`            | bool  | 仅使用 LNS（大邻域搜索）启发式，不进行完整树搜索。需配合时间限制使用。 |
| 输出控制   | `log_search_progress`     | bool  | 输出进度日志。为 `True` 时在控制台输出求解信息。             |
|            | `log_to_stdout`           | bool  | 是否输出到标准输出（默认 True）。可关闭后使用 `log_callback` 自定义处理。 |
| 随机性控制 | `random_seed`             | int   | 随机种子。固定种子使结果可重现。例如：`42`。                 |
|            | `randomize_search`        | bool  | 随机化搜索。为 `True` 时在搜索中引入随机性，可能找到不同解。 |

**CP-SAT 日志输出说明（深入解读）**

CP-SAT 的日志是理解求解过程、诊断性能瓶颈的关键。下面通过一个实际日志片段（来自 Primer）来解释各部分含义。

```text
Starting CP-SAT solver v9.10.4067
Parameters: max_time_in_seconds: 30 log_search_progress: true relative_gap_limit: 0.01
Setting number of workers to 16
```

*   **启动信息**：显示求解器版本、设置的参数和工作线程数。CP-SAT 默认使用所有核心，并根据核心数选择不同的子求解器组合。

```text
Initial optimization model '': (model_fingerprint: 0x1d316fc2ae4c02b1)
#Variables: 450 (#bools: 276 #ints: 6 in objective)
  - 342 Booleans in [0,1]
  - 12 in [0][10][20][30][40][50][60][70][80][90][100]
  ...
#kBoolOr: 30 (#literals: 72)
#kLinear1: 33 (#enforced: 12)
#kLinear2: 1'811
#kLinearN: 94 (#terms: 1'392)
```

*   **初始模型统计**：变量总数、布尔/整数变量数量、各变量的域分布、约束类型统计。例如 `- 12 in [21,57]` 表示有 12 个变量取值在 21 到 57 之间。`#kLinearN: 94 (#terms: 1'392)` 表示有 94 个线性约束，涉及 1392 个系数。这有助于检查模型是否符合预期，并发现潜在的膨胀。

```text
Starting presolve at 0.00s
  3.26e-04s  0.00e+00d  [DetectDominanceRelations]
  6.60e-03s  0.00e+00d  [PresolveToFixPoint] #num_loops=4 #num_dual_strengthening=3
  ...
[Symmetry] Graph for symmetry has 2'224 nodes and 5'046 arcs.
Presolve summary:
  - 54 affine relations were detected.
  - rule 'presolve: 33 unused variables removed.' was applied 1 time.
```

*   **预处理阶段**：CP-SAT 对模型进行简化，包括检测仿射关系、对称性、冗余变量等。例如，发现仿射关系（如 `x = 2y + 1`）并进行替换；通过对称性发现变量组可被对称破缺约束简化。预处理可以显著减小模型规模，但有时也会耗时。如果预处理时间过长，可考虑减少迭代次数或禁用某些选项。

```text
Presolved optimization model '': (model_fingerprint: 0xb4e599720afb8c14)
#Variables: 405 (#bools: 261 #ints: 6 in objective)
...
#kAtMostOne: 452 (#literals: 1'750)
```

*   **预求解后模型**：与初始模型对比，可见变量数减少，部分约束类型发生变化（如出现了 `kAtMostOne`）。这表明预求解成功简化了问题。

```text
Starting search at 0.05s with 16 workers.
11 full problem subsolvers: [core, default_lp, lb_tree_search, max_lp, no_lp, objective_lb_search, probing, pseudo_costs, quick_restart, quick_restart_no_lp, reduced_costs]
4 first solution subsolvers: [fj_long_default, fj_short_default, fs_random, fs_random_quick_restart]
9 incomplete subsolvers: [feasibility_pump, graph_arc_lns, graph_cst_lns, graph_dec_lns, graph_var_lns, rins/rens, rnd_cst_lns, rnd_var_lns, violation_ls]
3 helper subsolvers: [neighborhood_helper, synchronization_agent, update_gap_integral]
```

*   **并行 worker 分配**：CP-SAT 将不同的子求解器分配给各 worker，包括完整的树搜索求解器、首次可行解求解器、不完整的启发式（LNS）等。子求解器的选择依赖于问题和 worker 数量。

```text
#1       0.06s best:-0    next:[1,7125]   fj_short_default(batch:1 #lin_moves:0 #lin_evals:0 #weight_updates:0)
#2       0.07s best:1050  next:[1051,7125] fj_long_default(batch:1 #lin_moves:1'471 #lin_evals:2'102 #weight_updates:123)
#3       0.08s best:1051  next:[1052,7125] quick_restart_no_lp (fixed_bools=0/884)
#Bound   0.09s best:1051  next:[1052,1650] default_lp
#4       0.10s best:1052  next:[1053,1650] quick_restart_no_lp (fixed_bools=22/898)
...
#132    16.69s best:1450  next:[1451,1523] rnd_var_lns (d=0.88 s=657 t=0.10 p=0.54 stall=53 h=folio_rnd)
```

*   **搜索过程**：
    *   `#1`、`#2` 等表示找到的第 n 个可行解（incumbent solution）。
    *   `best:` 当前最优解的目标值。
    *   `next:[下界，上界]` 表示搜索的当前区间（对最小化问题，下界≤最优≤上界）。
    *   `#Bound` 表示下界更新（证明不可能有比该值更好的解）。
    *   括号内显示是哪个子求解器找到了解或改进了边界，以及一些内部信息（如 LNS 的破坏率 `d`、步骤 `s` 等）。
    *   通过观察解的改进频率、边界收紧的速度，可以判断求解的瓶颈：如果很长时间没有新解，可能是启发式不足；如果下界停滞，可能需要加强 LP 传播或添加切割。

```text
CpSolverResponse summary:
status: FEASIBLE
objective: 1450
best_bound: 1515
integers: 416
booleans: 857
conflicts: 0
branches: 370
propagations: 13193
walltime: 30.4515
deterministic_time: 306.548
gap_integral: 1292.47
solution_fingerprint: 0xf10c47f1901c2c16
```

*   **求解摘要**：状态、目标值、最佳边界、搜索统计、运行时间等。`gap_integral` 是间隙积分，反映收敛速度，值越小收敛越快。`solution_fingerprint` 可用于校验两次运行是否得到相同解。

**提示**：可以使用 [CP-SAT Log Analyzer](https://cpsat-log-analyzer.streamlit.app/) 在线分析日志，自动生成进度图并高亮关键信息。

---

### 📊 求解结果解析

**求解状态说明**

```python
from ortools.sat.python import cp_model

status = solver.solve(model)
if status == cp_model.OPTIMAL:
    print('✅ 找到最优解')
elif status == cp_model.FEASIBLE:
    print('⚠️  找到可行解（非最优）')
elif status == cp_model.INFEASIBLE:
    print('❌ 无可行解')
elif status == cp_model.MODEL_INVALID:
    print('❌ 模型无效')
else:
    print('❓ 求解状态未知')
```

**注意**：对于没有目标函数的模型，只要可行，CP-SAT 会返回 `OPTIMAL`（这可能会让新手困惑，但这是 SAT 求解器的常规行为）。

**获取求解统计信息**

```python
# 目标函数值 (属性)
obj_value = solver.objective_value

# 最佳下界（对最小化问题）(属性)
best_bound = solver.best_objective_bound

# 求解时间（秒）(属性)
wall_time = solver.wall_time

# 变量取值 (方法)
x_value = solver.value(x)

# 输出统计摘要 (方法)
print(solver.response_stats())
```

**提供启发式初始解（Hints）**

CP-SAT 允许通过 `add_hint()` 提供初始解，这可以加速搜索，尤其是当有好的启发式解时。

```python
from ortools.sat.python import cp_model

model = cp_model.CpModel()
x = model.new_int_var(0, 10, 'x')
y = model.new_int_var(0, 10, 'y')

# 添加提示：x 可能是 7，y 可能是 3
model.add_hint(x, 7)
model.add_hint(y, 3)

# ... 添加约束和目标 ...
solver = cp_model.CpSolver()
# 强制固定提示值（用于测试提示是否可行）
solver.parameters.fix_variables_to_their_hinted_value = True  
status = solver.solve(model)
```

**注意**：提示值必须可行（至少部分满足约束），否则会被忽略。可以通过 `fix_variables_to_their_hinted_value = True` 来检查提示是否可行（若不可行会返回 INFEASIBLE）。此外，提示在预求解后可能失效，可设置 `keep_all_feasible_solutions_in_presolve = True` 尝试保留。

---

## 第五部分：coptpy 完全指南：Python 接口详解与高级建模技巧

杉数求解器（Cardinal Optimizer, COPT）是上海杉数网络科技有限公司自主研发的高性能数学规划求解器，支持线性规划（LP）、整数规划（MILP）、二次规划（QP）、二阶锥规划（SOCP）、半定规划（SDP）以及非线性规划等多种问题类型。COPT 提供了丰富的编程接口，其中 coptpy 是官方提供的 Python 接口，它封装了 COPT 的高效求解引擎，并允许用户以 Python 语言的简洁性和灵活性构建复杂的优化模型。

### 5.1 安装与配置

coptpy 支持 Python 3.7 及以上版本（推荐 3.8–3.13）。安装方式有两种：通过 pip 在线安装，或通过 COPT 安装包手动安装。

**通过 pip 安装（推荐）**

在命令行执行：

```bash
pip install coptpy
```

如果已安装旧版本，升级到最新版：

```bash
pip install --upgrade coptpy
```

**通过 COPT 安装包安装**

首先下载 COPT 安装包（例如 CardinalOptimizer-8.0.3-win64.zip），解压后进入 `lib/python` 目录，执行：

```bash
python setup.py install
```

**注意**：若 COPT 安装在系统盘（如 C:\Program Files\copt80），请以管理员权限运行命令行。

**许可证配置**

coptpy 需要许可证文件 `license.dat` 和 `license.key` 才能运行。推荐将许可证文件放在用户目录下的 `copt` 文件夹中（例如 `C:\Users\username\copt\`），或者通过环境变量 `COPT_LICENSE_DIR` 指定许可证路径。详细配置请参考 COPT 用户手册第二章。

**验证安装是否成功：**

```python
import coptpy as cp
print(cp.COPT.__version__)
```

如果输出版本号且没有报错，说明安装成功。

### 5.2 coptpy 基础

coptpy 的建模过程通常包含以下步骤：

1.  创建环境（Envr）——管理许可证和全局设置。
2.  创建模型（Model）——容纳变量、约束、目标函数。
3.  添加变量（addVar / addVars）。
4.  添加约束（addConstr / addConstrs）。
5.  设置目标函数（setObjective）。
6.  设置求解参数（setParam）。
7.  求解（solve）。
8.  获取结果（objval, x, status 等）。

**示例：线性规划**

```python
import coptpy as cp
from coptpy import COPT

# 创建环境
env = cp.Envr()

# 创建模型
model = env.createModel("lp_ex1")

# 添加变量
x = model.addVar(lb=0.1, ub=0.6, name="x")
y = model.addVar(lb=0.2, ub=1.5, name="y")
z = model.addVar(lb=0.3, ub=2.8, name="z")

# 添加约束
model.addConstr(1.5*x + 1.2*y + 1.8*z  <= 2.6)
model.addConstr(0.8*x + 0.6*y + 0.9*z  >= 1.2)

# 设置目标函数
model.setObjective(1.2*x + 1.8*y + 2.1*z, sense=COPT.MAXIMIZE)

# 设置参数
model.setParam(COPT.Param.TimeLimit, 10.0)

# 求解
model.solve()

# 输出结果
if model.status == COPT.OPTIMAL:
    print("最优目标值：", model.objval)
    for var in [x, y, z]:
        print(f"{var.name} = {var.x}")
```

### 5.3 coptpy 核心类详解

coptpy 的核心类均位于 `coptpy` 模块中，主要包括：

*   **Envr**：环境类，负责许可证、全局配置和远程连接。
*   **Model**：模型类，包含变量、约束、目标以及求解方法。
*   **Var**：变量类，具有上下界、类型、取值等属性。
*   **Constraint**：线性约束类。
*   **LinExpr**：线性表达式，支持运算符重载。
*   **QuadExpr**：二次表达式，用于二次目标和约束。
*   **VarArray / ConstrArray**：变量/约束的数组容器，便于批量操作。
*   **MVar / MLinExpr / MQuadExpr**：矩阵建模相关类，支持多维数组运算。
*   **PsdVar / PsdExpr / PsdConstraint**：半定规划相关类。
*   **CallbackBase**：回调基类，用于实现自定义回调。
*   **NlExpr / nl 命名空间**：非线性表达式及函数。

下面详细介绍常用类及其方法。

**Envr 类**

Envr 是求解环境，可同时管理多个模型。主要方法：

*   `Envr(licDir=None, config=None)`：构造函数，可指定许可证路径或配置对象。
*   `createModel(name)`：创建一个新模型。
*   `close()`：关闭远程连接（用于浮动/集群许可）。
*   `bindNumaCpu(numaNode)` / `bindNumaMem(numaNode)`：绑定 CPU/内存到指定 NUMA 节点。
*   `setCpuAffinity(hexMask)`：设置 CPU 亲和度。

**Model 类**

Model 是建模和求解的核心类，包含数百个方法，此处列举常用方法：

*   **添加变量**：
    *   `addVar(lb=0.0, ub=COPT.INFINITY, obj=0.0, vtype=COPT.CONTINUOUS, name="", column=None)`：添加单个变量。
    *   `addVars(*indices, lb, ub, obj, vtype, nameprefix)`：批量添加变量，返回 `tupledict` 或 `VarArray`。
    *   `addMVar(shape, lb, ub, obj, vtype, nameprefix)`：添加矩阵变量（`MVar`），返回多维变量对象。
*   **添加约束**：
    *   `addConstr(lhs, sense=None, rhs=None, name="")`：添加线性约束，支持使用比较运算符直接构建。
    *   `addConstrs(generator, nameprefix)`：批量添加约束，例如 `model.addConstrs(x[i] + y[i] <= 1 for i in range(10))`。
    *   `addMConstr(A, x, sense, b, nameprefix)`：添加矩阵线性约束 `A x <= b`。
    *   `addQConstr(lhs, sense, rhs, name)`：添加二次约束。
    *   `addSOS(sostype, vars, weights=None)`：添加 SOS 约束。
    *   `addGenConstrIndicator(binvar, binval, lhs, sense, rhs, type, name)`：添加 Indicator 约束。
    *   `addCone(vars, ctype)`：添加二阶锥约束。
    *   `addExpCone(vars, ctype)`：添加指数锥约束。
    *   `addPsdConstr(expr, sense, rhs, name)`：添加半定约束。
*   **目标函数**：
    *   `setObjective(expr, sense)`：设置目标，`expr` 可为 `LinExpr`, `QuadExpr`, `MExpr`, `NlExpr`。
    *   `setMObjective(Q, c, constant, xQ_L, xQ_R, xc, sense)`：矩阵形式设置二次目标。
    *   `setObjectiveN(idx, expr, sense, priority, weight, abstol, reltol)`：设置多目标。
*   **求解与获取结果**：
    *   `solve()`：求解模型。
    *   `solveLP()`：按线性规划求解（忽略整数约束）。
    *   `getAttr(attrname)`：获取模型属性（如 `"Cols"`, `"Rows"`）。
    *   `getInfo(infoname, args)`：获取变量/约束的信息（如 `"LB"`, `"Value"`）。
    *   `getValues()`：获取所有变量取值。
    *   `getSlacks()` / `getDuals()`：获取约束松弛/对偶值。
    *   `getPoolObjVal(iSol)` / `getPoolSolution(iSol, vars)`：获取解池中的解。
*   **文件读写**：
    *   `read(filename)`：从文件读取模型（自动识别格式）。
    *   `write(filename)`：将模型写入文件。
*   **参数设置**：
    *   `setParam(paramname, newval)`：设置求解参数。
    *   `getParam(paramname)`：获取参数当前值。

**Var 类**

Var 对象代表一个决策变量，常用属性和方法：

*   `lb`, `ub`, `obj`, `vtype`：上下界、目标系数、变量类型（可读写）。
*   `x`：求解后的取值（只读）。
*   `rc`：Reduced cost（线性规划）。
*   `basis`：基状态。
*   `index`：变量在模型中的索引。
*   `getInfo(infoname)`：获取变量信息。
*   `setInfo(infoname, newval)`：设置信息（如修改上下界）。
*   `remove()`：从模型中删除变量。

**Constraint 类**

线性约束对象，常用属性和方法：

*   `name`：约束名称。
*   `pi`：对偶值。
*   `slack`：松弛量。
*   `basis`：基状态。
*   `index`：约束索引。
*   `getInfo(infoname)`：获取信息（`"LB"`, `"UB"`, `"Slack"` 等）。
*   `setInfo(infoname, newval)`：修改约束界限。
*   `remove()`：删除约束。

**表达式类：LinExpr 和 QuadExpr**

LinExpr 用于构建线性表达式，支持与 Var、常数进行加减乘除运算，且重载了比较运算符（`<=`, `>=`, `==`），可以直接生成约束构建器。

常用方法：

*   `LinExpr(expr)`：复制表达式。
*   `addTerm(var, coeff)` / `addTerms(vars, coeffs)`：添加项。
*   `addConstant(const)` / `setConstant(const)`：操作常数项。
*   `getValue()`：获取表达式在当前解下的值。
*   `clone()`：深拷贝。

QuadExpr 用于构建二次表达式，除了线性项，还包含二次项。可通过两个线性表达式相乘得到：`(x + y) * (z + w)` 返回 `QuadExpr`。

常用方法：

*   `addTerm(coeff, var1, var2=None)`：添加二次项（若 var2 为 None，则为线性项）。
*   `addQuadExpr(expr, mult)`：添加另一个二次表达式。
*   `getLinExpr()`：获取其中的线性部分。
*   `getCoeff(i)`, `getVar1(i)`, `getVar2(i)`：访问二次项。

**数组类：VarArray 和 ConstrArray**

VarArray 和 ConstrArray 分别用于存储一组变量或约束，提供批量操作的方法，常用于配合矩阵建模。

*   `pushBack(var)`：添加元素。
*   `getVar(i)` / `getConstr(i)`：获取指定索引的元素。
*   `size()`：获取元素个数。
*   `getAll()`：转换为 Python 列表。

这些数组类可以和 `addVars` / `addConstrs` 配合使用，返回的也是数组对象。

### 5.4 高级建模技巧

coptpy 提供了多种高级建模功能，帮助用户高效处理复杂优化问题。

**矩阵建模（MVar, MLinExpr, MQuadExpr）**

当模型变量规模大且结构规整时，使用矩阵建模可以显著简化代码并提高性能。coptpy 支持 `MVar`（多维变量数组）、`MLinExpr`（多维线性表达式）和 `MQuadExpr`（多维二次表达式），它们可以与 NumPy 数组无缝结合。

**创建矩阵变量：**

```python
mx = model.addMVar(shape=(3, 4), lb=0, ub=10, vtype=COPT.CONTINUOUS, nameprefix="x")
```

`shape` 可以是整数或元组，返回的 `MVar` 对象支持类似 NumPy 的索引和切片操作。

**构建线性表达式：**

```python
import numpy as np
A = np.array([[1, 2, 3, 4], [5, 6, 7, 8]])
expr = A @ mx   # 矩阵乘法，形状 (2, 4)  @ (4,) -> (2,)
expr2 = mx.sum(axis=0)  # 对第一维求和
```

**添加矩阵约束：**

```python
b = np.array([10, 20])
model.addMConstr(A, mx, 'L', b, nameprefix="con")  # 添加约束 A @ x <= b
```

也可以直接使用比较运算符：

```python
model.addConstrs(A @ mx <= b, nameprefix="con")
```

**设置矩阵目标：**

```python
c = np.array([1, 2, 3, 4])
model.setObjective(c @ mx, sense=COPT.MINIMIZE)
```

对于二次目标，可以使用 `setMObjective`：

```python
Q = np.diag([1, 2, 3, 4])
model.setMObjective(Q, None, 0, mx, mx, None, sense=COPT.MINIMIZE)
```

矩阵建模不仅语法简洁，而且底层利用 COPT 的 C++ 原生数组（`NdArray`）进行计算，避免了 Python 循环，效率更高。

**特殊约束**

* **SOS 约束**

  SOS（Special Ordered Set）约束用于限制一组变量中非零变量的个数及顺序。coptpy 支持 SOS1 和 SOS2：

  ```python
  model.addSOS(COPT.SOS_TYPE1, [x1, x2, x3], weights=[1, 2, 3])
  ```

* **Indicator 约束**

  Indicator 约束表达逻辑关系：若二进制变量取某值，则线性约束成立。coptpy 支持三种类型：If-Then, Only-If, If-and-Only-If。

  ```python
  # If-Then: 如果 x == 1，则 y + 2*z >= 3
  model.addGenConstrIndicator(x, True, y + 2*z >= 3)
  
  # Only-If: 如果 y + 2*z <= 3，则 x == 0
  model.addGenConstrIndicator(x, False, y + 2*z <= 3, type=COPT.INDICATOR_ONLYIF)
  ```

  也可以使用重载运算符 `>>` 和 `<<`：

  ```python
  model.addConstr((x == 1) >> (y + 2*z >= 3))
  model.addConstr((x == 0) << (y + 2*z <= 3))
  ```

* **锥约束**

  coptpy 支持二阶锥（SOCP）、指数锥和仿射锥约束。

  ```python
  model.addCone([t, x, y], COPT.CONE_QUAD)   # 标准二阶锥 t >= sqrt(x^2 + y^2)
  model.addCone([x, y, z, w], COPT.CONE_RQUAD)  # 旋转二阶锥 2*x*y >= z^2 + w^2
  ```

  指数锥（常用于几何规划等）：

  ```python
  model.addExpCone([u, 1, v], COPT.EXPCONE_PRIMAL)   # u >= exp(v)
  ```

  仿射锥：将线性表达式或半定表达式放入锥中：

  ```python
  model.addAffineCone(cp.vstack(c@x + d, A@x + b), ctype=COPT.CONE_QUAD)
  ```

* **半定规划（SDP）**

  coptpy 支持半定变量和半定约束。半定变量用 `PsdVar` 表示，对称矩阵用 `SymMatrix` 表示。

  添加半定变量：

  ```python
  X = model.addPsdVar(dim=3, name="X")
  ```

  构建半定表达式：

  ```python
  C = model.addOnesMat(3)  # 全 1 对称矩阵
  expr = C * X             # 半定项
  expr += x + y            # 添加线性项
  ```

  添加半定约束：

  ```python
  model.addPsdConstr(expr == 1, name="psd_c")
  ```

  半定约束可以是等式或不等式，支持单边和双边。

**非线性建模**

coptpy 支持两种非线性建模方式：显式表达式和回调接口。

* **显式非线性表达式**

  利用 `nl` 命名空间中的函数（如 `nl.sin`, `nl.exp`, `nl.log`, `nl.pow` 等），可以构建复杂的非线性表达式。

  ```python
  from coptpy import nl
  
  expr = nl.sin(x) * nl.exp(y) + nl.log(z)
  model.setObjective(expr, sense=COPT.MINIMIZE)
  
  constr = nl.sqrt(x) + y**2 <= 5
  model.addNlConstr(constr)
  ```

  **注意**：显式非线性表达式目前仅支持连续模型，且要求问题为凸或可处理为凸（通过参数设置）。

* **回调接口（用户提供导数信息）**

  对于难以用显式表达式描述的非线性模型，用户可以实现 `NlpCallbackBase` 回调类，在给定点计算目标值、约束值及导数，然后通过 `loadNlData` 注册到模型中。

  ```python
  class MyNlpCallback(NlpCallbackBase):
      def EvalObj(self, xdata, outdata):
          x = xdata[0]; y = xdata[1]
          outdata[0] = x**2 + y**2   # 目标函数
      def EvalGrad(self, xdata, outdata):
          outdata[0] = 2*xdata[0]
          outdata[1] = 2*xdata[1]
      # 可以继续实现 EvalCon, EvalJac, EvalHess...
  
  cb = MyNlpCallback()
  model.loadNlData(nCols=2, nRows=1, sense=COPT.MINIMIZE,
                   nGrad=2, idxGrad=[0,1],
                   nJac=2, idxJacRow=[0,0], idxJacCol=[0,1],
                   nHess=2, idxHessRow=[0,1], idxHessCol=[0,1],
                   colLower=[-1,-1], colUpper=[1,1],
                   rowLower=[0], rowUpper=[0],
                   initX=[0,0], evalType=-1, cb=cb)
  model.solve()
  ```

  回调接口适用于非凸问题、自定义函数或需要外部数值计算的场景。

**多目标优化**

coptpy 支持分层法（优先级）和加权组合法处理多目标线性规划。

**设置多目标：**

```python
model.setObjectiveN(0, x + y, sense=COPT.MINIMIZE, priority=2, weight=1)
model.setObjectiveN(1, x - y, sense=COPT.MAXIMIZE, priority=1, weight=2)
```

求解多目标模型时，COPT 会按优先级从高到低依次优化，相同优先级的目标按加权组合。

**获取多目标结果：**

```python
obj0 = model.getAttrN(0, "BestObj")
obj1 = model.getAttrN(1, "BestObj")
```

**回调函数（Callbacks）**

回调函数允许用户在求解过程中监控进度、添加惰性约束或割平面、注入可行解等。coptpy 通过继承 `CallbackBase` 类实现。

**定义回调类：**

```python
class MyCallback(CallbackBase):
    def callback(self):
        if self.where() == COPT.CBCONTEXT_MIPSOL:
            # 获取当前可行解
            sol = self.getSolution(vars)
            # 检查是否需要添加惰性约束
            if violated(sol):
                self.addLazyConstr(x + y <= 1)
```

**注册回调：**

```python
cb = MyCallback()
model.setCallback(cb, COPT.CBCONTEXT_MIPSOL | COPT.CBCONTEXT_MIPNODE)
```

回调中可用的方法包括 `getInfo`, `getRelaxSol`, `getIncumbent`, `addUserCut`, `addLazyConstr`, `setSolution`, `loadSolution` 等。

**不可行模型处理**

当模型不可行时，COPT 可计算不可约不一致子系统（IIS）或可行化松弛。

**计算 IIS：**

```python
model.computeIIS()
model.writeIIS("model.iis")
# 获取变量/约束的 IIS 状态
iis_vars = model.getVarLowerIIS(vars)
iis_cons = model.getConstrLowerIIS(cons)
```

**可行化松弛：**

```python
# 简化模式：松弛所有变量和约束
model.feasRelaxS(vrelax=True, crelax=True)

# 精细模式：指定松弛项和惩罚因子
model.feasRelax(vars, lbpen, ubpen, constrs, rhspen, uppen)
```

可行化松弛后，可通过 `getInfo(COPT.Info.RelaxValue)` 等获取松弛后的解。

**整数规划初始解**

为整数规划提供好的初始解可加速求解。coptpy 提供多种方式设置初始解：

**通过文件读取：**

```python
model.readMst("initial.mst")
```

**直接设置：**

```python
model.setMipStart(x, 1.0)
model.setMipStart([y, z], [2.0, 3.0])
model.loadMipStart()   # 加载所有已设置的值
```

通过参数 `MipStartMode` 控制初始解的使用方式（完整/不完整解的处理）。

**参数调优工具**

COPT 内置参数调优工具，可以自动寻找更好的参数组合。coptpy 中通过 `tune()` 方法启动：

```python
model.setParam(COPT.Param.TuneTimeLimit, 300)  # 调优时间限制
model.tune()
# 获取调优结果
best_idx = 0
model.loadTuneParam(best_idx)   # 加载最佳参数
model.solve()
```

调优结果可以通过 `getAttr(COPT.IntAttr.TuneResults)` 获取结果数量。

**GPU 加速与并行设置**

COPT 支持使用 GPU 加速线性规划和内点法求解。coptpy 中通过参数启用：

```python
model.setParam(COPT.Param.GPUMode, 1)      # 启用 GPU
model.setParam(COPT.Param.GPUDevice, 0)    # 指定 GPU 设备编号
```

并行线程数可通过 `Threads` 等参数控制：

```python
model.setParam(COPT.Param.Threads, 8)      # 全局线程数
model.setParam(COPT.Param.BarThreads, 4)   # 内点法线程数
model.setParam(COPT.Param.MipTasks, 16)    # MIP 任务数
```

### 5.5 日志详解

COPT 的求解日志是监控求解过程、诊断性能问题的关键工具。以下是一个 MIP 求解日志的典型片段及解读。

**求解开始信息**

```text
Using Cardinal Optimizer v8.0.3 on Windows (25H2 Build 26200 - x86_64)
The CPU model is Intel(R) Core(TM) i7-14650HX
Hardware has 16 physical cores and 24 logical cores. Using instruction set X86_AVX2 (10)
Minimizing a MIP problem
```

*   **版本与平台**：显示 COPT 版本 8.0.3，运行在 Windows 系统，x86_64 架构。
*   **硬件信息**：CPU 型号、物理核心数（16）、逻辑核心数（24）。COPT 将利用这些核心并行计算，指令集为 AVX2。
*   **问题类型**：正在最小化一个 MIP（混合整数规划）问题。

**原始模型规模**

```text
The original problem has:
    1262604 rows, 282884 columns and 2796822 non-zero elements
    273656 binaries and 9228 integers
    19551 indicators
```

*   `rows`：约束数（1,262,604）。
*   `columns`：变量数（282,884）。
*   `non-zero elements`：系数矩阵非零元个数（2,796,822）。
*   `binaries`：二进制变量数（273,656）。
*   `integers`：整数变量数（9,228），注意二进制变量是整数变量的子集，此处统计的是除二进制外的整数变量。
*   `indicators`：Indicator 约束数（19,551），表明模型包含大量逻辑约束。

**预求解后模型规模**

```text
The presolved problem has:
    191525 rows, 54736 columns and 525869 non-zero elements
    50053 binaries and 3290 integers
    1338 indicators
```

预求解通过合并冗余约束、固定变量、系数化简等手段大幅缩减了模型规模：约束数降至约 19 万（减少 84%），变量数降至约 5.5 万（减少 80%），非零元降至约 52 万（减少 81%）。预求解是 MIP 求解的关键步骤，能显著提升后续分支定界效率。

**问题数值特征统计**

```text
Problem info:
    Range of matrix coefficients:    [1e+00,6e+04]
    Range of rhs coefficients:       [1e+00,6e+04]
    Range of bound coefficients:     [1e+00,6e+04]
    Range of cost coefficients:      [3e+00,2e+03]
    Density of cost:                     0.1%
```

*   `matrix coefficients`：系数矩阵中元素的范围 1 到 60,000，量级跨度不大，数值稳定性较好。
*   `rhs coefficients`：约束右端项范围。
*   `bound coefficients`：变量界限范围。
*   `cost coefficients`：目标函数系数范围。
*   `Density of cost`：目标函数系数的密度（非零比例），此处为 0.1%，说明目标函数非常稀疏。

**分支切割法求解过程日志**

```text
     Nodes    Active  LPit/n  IntInf     BestBound  BestSolution     Gap   Time
         0         1      --       0 -4.958900e+04            --     Inf 19.22s
H        0         1      --       0 -4.958900e+04  2.878000e+04 158.04% 21.36s
...
```

**日志表格各列含义：**

*   `Nodes`：已处理过的节点数（分支定界树中已求解 LP 松弛的节点）。
*   `Active`：尚未被搜索的叶子节点个数（活跃节点数）。
*   `LPit/n`：每个节点平均单纯形迭代次数（`--` 表示根节点尚未求解完）。
*   `IntInf`：当前 LP 松弛解中不满足整数约束的变量个数（用于指导分支）。
*   `BestBound`：当前最优的目标下界（对最小化问题而言，是可行解目标值的下界）。
*   `BestSolution`：当前最优的可行解目标值。
*   `Gap`：相对间隙 = (BestSolution - BestBound) / |BestSolution|（若 BestSolution 存在），当间隙小于参数 `RelGap`（默认 1e-4）时停止求解。
*   `Time`：累计求解时间（秒）。

**日志中特殊标记的含义：**

*   `H`：通过启发式（Heuristic）找到一个新的可行解。
*   `*****`：通过分支（Branching）求解子问题找到一个新的可行解。

**日志片段解析**

```text
         0         1      --       0 -4.958900e+04            --     Inf 19.22s
```

根节点求解前，无可行解，BestBound 为 -4.9589e4（可能是通过对偶或线性松弛得到的下界），BestSolution 为 `--`，Gap 为无穷大。

```text
H        0         1      --       0 -4.958900e+04  2.878000e+04 158.04% 21.36s
```

启发式找到第一个可行解，目标值 28780，间隙 158.04%（说明下界远小于可行解，间隙很大）。

```text
         0         1      --    2437  1.766000e+03  2.741500e+04  93.75% 23.38s
```

根节点 LP 松弛求解完成，得到下界 1766（较之前大幅提升），当前最好解仍为 27415，间隙 93.75%。`IntInf=2437` 表示松弛解中有 2437 个整数变量不满足整数性。

后续日志中，BestBound 逐步提升，BestSolution 通过启发式和分支不断改进，间隙逐渐缩小。

**关键转折点：**

```text
H     3057      2393   986.4     151  1.766000e+03  4.117000e+03  58.37%   401s
H     3539      2821    1009     441  1.766000e+03  3.274000e+03  47.65%   446s
H     4074      3275   967.6     243  1.766000e+03  2.836000e+03  39.56%   476s
```

通过启发式连续改进可行解，从 4117 降至 3274 再降至 2836。

```text
*     7212      2148   740.3       0  1.766000e+03  2.829000e+03  39.41%   676s
*     7430      2267   720.1       0  1.766000e+03  2.288000e+03  25.09%   679s
*     7463      1977   717.0       0  1.766000e+03  2.053000e+03  16.51%   679s
```

通过分支找到三个改进解，目标值从 2829 降至 2288 再降至 2053，间隙从 39.41% 降至 16.51%。注意 `IntInf` 为 0 表示当前节点 LP 松弛解已是整数可行解（但可能不是全局最优，因为下界仍为 1766）。

```text
H     7722      1804   695.9     109  1.766000e+03  1.795000e+03  4.513%   683s
```

启发式找到接近最优的解 1795，间隙降至 4.5%。

**求解结果汇总**

```text
Best solution   : 1795.000000000
Best bound      : 1766.000000000
Best gap        : 4.5125%
Solve time      : 683.90
Solve node      : 7741
MIP status      : solved
Solution status : integer optimal (relative gap limit 0.0001)

Violations      :     absolute     relative
    bounds      :            0            0
    rows        :            0            0
    integrality :            0
    indicators  :            0
```

*   `Best solution`：最优可行解目标值 1795。
*   `Best bound`：最优下界 1766。
*   `Best gap`：相对间隙 4.5125%，超过了参数 `RelGap` 默认值 1e-4，但求解器为何停止？观察状态为 `solved`，可能因其他终止条件（如时间限制）或参数 `AbsGap` 设为 258 导致绝对间隙满足条件？实际绝对间隙 = 1795-1766 = 81 < 258，绝对间隙满足，因此求解器判定最优。
*   `Solve time`：总求解时间 683.9 秒。
*   `Solve node`：处理节点数 7741。
*   `MIP status`：`solved` 表示求解正常结束。
*   `Solution status`：`integer optimal (relative gap limit 0.0001)` 虽实际间隙未达 0.01%，但状态仍为 optimal（可能因绝对间隙满足或参数设置）。
*   `Violations`：显示最优解对约束、变量界限、整数性、Indicator 约束的违反程度均为 0，说明解严格可行。

---

## 第六部分：优化建模技巧与实践（基于 CP-SAT 经验）

### 1. 减少不必要的变量

*   **为什么重要**：每个变量都会增加搜索空间维度，降低求解速度。CP-SAT 内部会将整数变量转换为布尔变量（通过顺序编码），但变量数量仍然直接影响求解规模。
*   **具体做法**：
    *   用已有变量表达式代替新变量（例如，用线性组合代替中间变量）。
    *   合并含义相似的变量。
    *   使用中间计算而非变量存储中间值。
    *   **示例**：如果只需要知道是否生产（0/1），就不要同时定义生产数量变量；如果必须使用数量，尽量限制其取值范围。

### 2. 最小化变量的上下限

*   **为什么重要**：边界越紧，分支定界剪枝越有效，CP-SAT 的域传播也越强。
*   **具体做法**：
    *   根据业务逻辑收紧边界（例如，不可能生产超过 1000 件，就设上限 1000）。
    *   动态计算可能范围（比如通过预先计算最大可行产量）。
    *   使用约束传播后的边界（可先求解一次可行性模型获得收紧的域，参见 Primer 中的 `fill_tightened_domains_in_response` 参数）。

### 3. 一个约束一个约束地增加（迭代建模）

*   **为什么重要**：便于调试，快速定位问题约束。CP-SAT 的日志可以帮助判断哪个约束导致问题。
*   **具体做法**：
    *   **基础模型**：只加核心约束（如物料平衡），验证可行性。
    *   **逐步增强**：每次添加 1-2 个新约束类型（如设备容量、换型时间），每个阶段检查解的合理性。
    *   **验证中间结果**：如果模型变无解，立即回退并检查新约束。

### 4. 大型排程的小范围测试策略

*   **为什么重要**：避免长时间运行后才发现模型错误。
*   **具体做法**：
    *   **时间切片**：先排 1 天的计划，再扩展到 1 周。
    *   **资源子集**：先用 10 台机器测试，再扩展到 100 台。
    *   **产品抽样**：选 5 种代表性产品测试，再扩展到全部。
    *   **参数简化**：用简化业务规则（如忽略换型时间）验证逻辑。

### 5. 结果验证与增量开发

*   **具体做法**：
    *   **完整性检查**：解是否满足所有硬约束？可以通过独立验证函数检查。
    *   **合理性检查**：产能利用率、等待时间等指标是否合理？
    *   **边界测试**：极值情况下的表现（如需求为零、产能无限）。
    *   **对比基准**：与简单规则（如先到先服务）或历史方案对比。

### 6. 性能优化技巧（CP-SAT 高级）

*   **高级策略**：
    *   **对称性破除**：对相同机器/产品添加顺序约束（例如，强制相同机器的生产顺序按索引排列）。
    *   **松弛模型**：先用线性规划松弛（可通过 CP-SAT 的 LP propagator 或外部 LP 求解器）快速获得下界。
    *   **启发式初始解**：提供好的初始解（hints）加速求解。CP-SAT 接受 `add_hint()`，但要求提示值可行。
    *   **求解器参数调优**：根据问题类型调整参数，例如：
        *   `num_search_workers`：并行线程数（建议设为物理核心数）。
        *   `max_time_in_seconds`：时间限制。
        *   `relative_gap_limit`：相对最优间隙提前停止。
        *   `log_search_progress`：开启日志以分析瓶颈。
        *   `use_energetic_reasoning_in_no_overlap_2d`：对 2D 排程启用能量推理，可能加速。
*   **线性表达式优化**：
    *   当约束中包含大量变量的求和时（例如 `sum(var_list)`），使用 `cp_model.LinearExpr.sum(var_list)` 比 Python 内置的 `sum()` 性能更好。

### 7. 调试与日志记录

*   **建议做法**：
    *   记录每次添加约束的影响（比如约束数量、变量数量）。
    *   输出中间可行解的关键指标（如完工时间、库存水平）。
    *   使用求解器日志分析瓶颈（参见第三部分对 CP-SAT 日志的详细解读）。
    *   **模型验证**：在求解前使用 `model.validate()` 检查模型有效性，使用 `model.model_stats()` 查看模型统计信息。
    *   **不可行分析**：当模型不可行时，使用 `model.add_assumption()` 添加假设，求解后通过 `solver.sufficient_assumptions_for_infeasibility()` 获取导致不可行的核心约束子集。

优化建模是迭代过程，不是一次性任务。从简单开始，逐步复杂化，持续验证，这是应对复杂排程问题的稳健策略。

---

## 第七部分：实战应用、问题解决与学习资源

### 🚀 OR-Tools 两阶段生产排程示例

下面是一个两阶段生产排程的 CP-SAT 模型示例，展示如何定义变量、添加约束和目标，并提取结果。该示例考虑了产品需求、机器容量、工序间库存等基本要素。

**注意**：本示例已更新为 OR-Tools v9+ 推荐的 **snake_case** API 风格，并修复了原代码中的拼写错误。

```python
"""
生产排程优化模型示例（两阶段流水车间）
使用 Google OR-Tools 的 CP-SAT 求解器
"""

from ortools.sat.python import cp_model
import pandas as pd
import numpy as np
from datetime import datetime
import os

class TwoStageProductionScheduler:
    """
    两阶段生产排程模型

    目标：最小化订单延迟惩罚
    约束：
        - 每个班次每台机器最多生产一个产品
        - 物料平衡：Stage 2 的消耗不能超过 Stage 1 的库存
        - 最终库存需尽可能满足需求（通过惩罚延迟）
    """

    def __init__(self, num_products=4, num_machines_stage1=3, num_machines_stage2=3,
                 num_shifts=30, demand_per_product=None):
        self.num_products = num_products
        self.num_machines_stage1 = num_machines_stage1
        self.num_machines_stage2 = num_machines_stage2
        self.num_shifts = num_shifts
        self.demand = demand_per_product if demand_per_product else [500, 300, 400, 200]

        # 假设每个班次固定产量 100 件
        self.production_rate = 100

        # 初始库存（示例）
        self.initial_inv_stage1 = [100, 80, 120, 60]
        self.initial_inv_stage2 = [50, 40, 60, 30]

        # 创建模型
        self.model = cp_model.CpModel()
        self.solver = cp_model.CpSolver()

        # 设置求解参数 (使用 snake_case)
        self.solver.parameters.max_time_in_seconds = 300   # 5 分钟
        self.solver.parameters.num_search_workers = 4
        self.solver.parameters.log_search_progress = True 

        # 存储变量
        self.produce_s1 = {}   # (product, machine, shift) -> BoolVar
        self.produce_s2 = {}
        self.inv_s1 = {}       # (product, shift) -> IntVar
        self.inv_s2 = {}
        self.shortage = {}     # product -> IntVar

    def build_model(self):
        """构建优化模型"""
        # 索引
        P = range(self.num_products)
        M1 = range(self.num_machines_stage1)
        M2 = range(self.num_machines_stage2)
        T = range(self.num_shifts)

        # ---------- 决策变量 ----------
        # 生产标识变量
        for p in P:
            for m in M1:
                for t in T:
                    self.produce_s1[(p, m, t)] = self.model.new_bool_var(f'prod_s1_p{p}_m{m}_t{t}')
            for m in M2:
                for t in T:
                    self.produce_s2[(p, m, t)] = self.model.new_bool_var(f'prod_s2_p{p}_m{m}_t{t}')

        # 库存变量（每个产品每个班次末）
        for p in P:
            for t in T:
                self.inv_s1[(p, t)] = self.model.new_int_var(0, 10000, f'inv_s1_p{p}_t{t}')
                self.inv_s2[(p, t)] = self.model.new_int_var(0, 10000, f'inv_s2_p{p}_t{t}')

        # 短缺变量（用于目标函数）
        for p in P:
            self.shortage[p] = self.model.new_int_var(0, self.demand[p], f'shortage_p{p}')

        # ---------- 约束 ----------
        # 1. 每台机器每个班次最多生产一个产品
        for m in M1:
            for t in T:
                # 使用 LinearExpr.sum 优化大规模求和
                self.model.add(cp_model.LinearExpr.sum(self.produce_s1[(p, m, t)] for p in P) <= 1)
        for m in M2:
            for t in T:
                self.model.add(cp_model.LinearExpr.sum(self.produce_s2[(p, m, t)] for p in P) <= 1)

        # 2. 物料平衡
        for p in P:
            # Stage 1 库存平衡
            for t in T:
                total_prod_s1 = sum(self.produce_s1[(p, m, t)] * self.production_rate for m in M1)
                if t == 0:
                    self.model.add(self.inv_s1[(p, t)] == self.initial_inv_stage1[p] + total_prod_s1)
                else:
                    self.model.add(self.inv_s1[(p, t)] == self.inv_s1[(p, t-1)] + total_prod_s1)

            # Stage 2 库存平衡
            for t in T:
                total_prod_s2 = sum(self.produce_s2[(p, m, t)] * self.production_rate for m in M2)
                if t == 0:
                    self.model.add(self.inv_s2[(p, t)] == self.initial_inv_stage2[p] + total_prod_s2)
                else:
                    self.model.add(self.inv_s2[(p, t)] == self.inv_s2[(p, t-1)] + total_prod_s2)

            # 工序间约束：Stage 2 生产不能超过前一个班次 Stage 1 的库存
            for t in T:
                if t > 0:
                    total_prod_s2_t = sum(self.produce_s2[(p, m, t)] * self.production_rate for m in M2)
                    self.model.add(total_prod_s2_t <= self.inv_s1[(p, t-1)])

        # 3. 最终库存与需求的关系：短缺量 = max(0, 需求 - 最终库存)
        for p in P:
            final_inv = self.inv_s2[(p, self.num_shifts-1)]
            self.model.add(self.shortage[p] >= self.demand[p] - final_inv)

        # ---------- 目标函数 ----------
        # 最小化总短缺量（可乘以惩罚系数）
        total_shortage = sum(self.shortage[p] for p in P)
        self.model.minimize(total_shortage)

    def solve(self):
        """求解模型"""
        print("开始求解...")
        # 可选：在求解前验证模型
        # print(self.model.validate()) 
        status = self.solver.solve(self.model)
        if status == cp_model.OPTIMAL:
            print("找到最优解！")
        elif status == cp_model.FEASIBLE:
            print("找到可行解（非最优）")
        else:
            print(f"求解失败，状态码：{status}")
            # 如果不可行，尝试分析假设
            if status == cp_model.INFEASIBLE:
                print("尝试分析不可行原因...")
                # 需要先添加 assumption 才能使用此功能
            return False
        return True

    def extract_schedule(self):
        """提取生产计划"""
        if not hasattr(self, 'solver') or self.solver.objective_value is None:
            print("请先求解模型")
            return None

        schedule_s1 = []
        schedule_s2 = []

        # 遍历所有生产变量，提取值为 1 的
        for (p, m, t), var in self.produce_s1.items():
            if self.solver.value(var):
                schedule_s1.append({
                    'product': p,
                    'machine_stage1': m,
                    'shift': t,
                    'quantity': self.production_rate
                })

        for (p, m, t), var in self.produce_s2.items():
            if self.solver.value(var):
                schedule_s2.append({
                    'product': p,
                    'machine_stage2': m,
                    'shift': t,
                    'quantity': self.production_rate
                })

        df_s1 = pd.DataFrame(schedule_s1)
        df_s2 = pd.DataFrame(schedule_s2)

        # 计算短缺量
        shortages = [self.solver.value(self.shortage[p]) for p in range(self.num_products)]

        return {
            'stage1': df_s1,
            'stage2': df_s2,
            'objective': self.solver.objective_value,
            'shortages': shortages
        }

def run_example():
    """运行示例"""
    scheduler = TwoStageProductionScheduler(
        num_products=4,
        num_machines_stage1=3,
        num_machines_stage2=3,
        num_shifts=20,
        demand_per_product=[500, 300, 400, 200]
    )
    scheduler.build_model()
    if scheduler.solve():
        result = scheduler.extract_schedule()
        print("\n=== 求解结果 ===")
        print(f"目标值（总短缺量）: {result['objective']}")
        print(f"各产品短缺量：{result['shortages']}")
        print("\nStage 1 生产计划:")
        if not result['stage1'].empty:
            print(result['stage1'].to_string(index=False))
        else:
            print("无生产任务")
        print("\nStage 2 生产计划:")
        if not result['stage2'].empty:
            print(result['stage2'].to_string(index=False))
        else:
            print("无生产任务")

if __name__ == "__main__":
    run_example()
```

### 🚀 COPT 使用 GPU 示例

```python
import coptpy as cp
from coptpy import COPT

# 创建求解环境
env = cp.Envr()

# 创建模型
model = env.createModel("lp_gpu_example")

# 添加变量（x, y, z），设置上下界
x = model.addVar(lb=0.1, ub=0.6, name="x")
y = model.addVar(lb=0.2, ub=1.5, name="y")
z = model.addVar(lb=0.3, ub=2.8, name="z")

# 添加约束
model.addConstr(1.5 * x + 1.2 * y + 1.8 * z  <= 2.6)
model.addConstr(0.8 * x + 0.6 * y + 0.9 * z  >= 1.2)

# 设置目标函数（最大化）
model.setObjective(1.2 * x + 1.8 * y + 2.1 * z, sense=COPT.MAXIMIZE)

# ========== GPU 相关参数设置 ==========
# 选择求解线性规划的算法：6 表示一阶算法 PDLP（支持 GPU）
model.setParam(COPT.Param.LpMethod, 6)
# 启用 GPU 模式：1 尝试使用 GPU 标准模式；2 高性能模式（可能更高内存占用）
model.setParam(COPT.Param.GPUMode, 1)
# 可选：指定使用哪块 GPU（如果有多块），默认 -1 自动选择
# model.setParam(COPT.Param.GPUDevice, 0)

# 可选：设置求解时间限制
model.setParam(COPT.Param.TimeLimit, 10.0)

# 求解模型
model.solve()

# 输出结果
if model.status == COPT.OPTIMAL:
    print(f"目标函数值：{model.objval:.6f}")
    print("变量取值:")
    for var in model.getVars():
        print(f"  {var.name} = {var.x:.6f}")
else:
    print("未找到最优解")
```

### ❗ OR-Tools 运行报错：OSError: [WinError 127] 找不到指定的程序。

```python
from ortools.sat.python import cp_model
from ortools.linear_solver import pywraplp
```

在运行 ortools 导入语句的 Python 程序时出现错误：

```text
Traceback (most recent call last):
  File "D:\AiProject\zj\3_production_scheduling_pywraplp.py", line 7, in <module>
    from ortools.linear_solver import pywraplp
  File "D:\anaconda3\Lib\site-packages\ortools\__init__.py", line 72, in <module>
    _load_ortools_libs()
    ~~~~~~~~~~~~~~~~~~^^
  File "D:\anaconda3\Lib\site-packages\ortools\__init__.py", line 67, in _load_ortools_libs
    WinDLL(dll_path)
    ~~~~~~^^^^^^^^^^
  File "D:\anaconda3\Lib\ctypes\__init__.py", line 390, in __init__
    self._handle = _dlopen(self._name, mode)
                   ~~~~~~~^^^^^^^^^^^^^^^^^^
OSError: [WinError 127] 找不到指定的程序。
```

**两个解决方法**

1. **安装 Microsoft Visual C++ Redistributable**

   这个错误通常是由于缺少必要的 C++ 运行库导致的。OR-Tools 需要这些库来加载其原生 DLL 文件。

   **解决方案：**

   *   访问微软官网下载并安装最新版的 [Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)
   *   建议同时安装 x86 和 x64 版本
   *   安装完成后重启计算机

2. **将 ortools 导入语句放在程序第一行**

   在某些情况下，Python 的导入顺序会影响库的加载。将 ortools 导入放在程序的最开始可以解决这个问题。

   **修改前的代码：**

   ```python
   # 其他导入
   import numpy as np
   import pandas as pd
   
   # OR-Tools 导入在中间
   from ortools.sat.python import cp_model
   from ortools.linear_solver import pywraplp
   
   # 其他代码...
   ```

   **修改后的代码：**

   ```python
   # 将 OR-Tools 导入放在第一行
   from ortools.sat.python import cp_model
   from ortools.linear_solver import pywraplp
   
   # 其他导入放在后面
   import numpy as np
   import pandas as pd
   
   # 其他代码...
   ```

---

### 📚 学习资源

| 类别               | 工具/项目名称                      | 特点与用途                                                   | 参考信息与链接                                               |
| :----------------- | :--------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 数学规划求解器     | COPT (杉数)                        | 国产商业求解器，性能优异。                                   | 官网：[杉数科技 COPT](https://www.shanshu.ai/copt)           |
| 商业               | Gurobi                             | 业界领先的商业求解器，高性能。                               | 官网：[Gurobi Optimization](https://www.gurobi.com)          |
| 闭源               | MindOpt (阿里)                     | 阿里达摩院推出的优化求解器。                                 | 官网：[阿里云 MindOpt](https://opt.aliyun.com)               |
| 商业               | IBM ILOG CPLEX Optimization Studio | 经典的商业优化求解器，支持 LP、MILP、QP、CP 等。             | 官网：[IBM CPLEX](https://www.ibm.com/analytics/cplex-optimizer) |
| 启发式算法库       | scikit-opt                         | 提供遗传算法、模拟退火等启发式算法，用于快速寻找优质可行解。 | [scikit-opt 中文文档](https://scikit-opt.github.io/scikit-opt/#/zh/README) |
| 建模语言与工具     | MiniZinc                           | 一种声明式的高层建模语言，可将模型与多种后端求解器（包括 OR-Tools）解耦。适合快速原型验证和算法研究。 | [MiniZinc 官网](https://www.minizinc.org/)                   |
| 领域专用框架       | PyJobShop                          | 一个专门用于作业车间调度问题（JSP）建模、求解和可视化的 Python 库，基于 OR-Tools 等求解器构建，提供标准案例和评估工具。 | [PyJobShop 文档](https://pyjobshop.org/stable/setup/intro_to_cp.html) |
| 教程与学习资源     | cpsat-primer                       | 一个专注于 Google OR-Tools CP-SAT 求解器的入门教程和代码示例库，包含大量带注释的实战案例，是深入掌握 CP-SAT 的绝佳补充材料。 | [GitHub 仓库：cpsat-primer](https://github.com/d-krupke/cpsat-primer) |
| 官方核心资源       | Google OR-Tools                    | 本指南核心工具，开源运筹学库。                               | 官方文档：[developers.google.com/optimization](https://developers.google.com/optimization) <br> GitHub：[google/or-tools](https://github.com/google/or-tools) <br> 社区：[Stack Overflow](https://stackoverflow.com/questions/tagged/or-tools) |
| 基准测试与日志分析 | CP-SAT Log Analyzer                | 在线工具，用于可视化 CP-SAT 日志，自动生成搜索进度图，帮助理解求解瓶颈。 | [CP-SAT Log Analyzer](https://cpsat-log-analyzer.streamlit.app/) |
| COPT 官方文档      | 杉数求解器用户手册                 | 详细的 COPT 使用文档，包括参数说明、API 参考等。             | [COPT 用户手册](https://guide.coap.online/copt/)             |

本篇深入介绍了 OR-Tools 和 coptpy 的实战用法，从安装到建模技巧，再到两阶段生产排程案例，为 APS 系统的实现提供了坚实的代码基础。接下来，我们将进一步探索 GPU 加速的优化世界，详细介绍 NVIDIA cuOpt 的 LP、QP、MILP 建模与求解能力，帮助您在大规模优化场景下获得更优的性能表现。
