---
title: PyTorch详解-优化模块
date: 2026-01-03 08:03:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](PyTorch详解-优化模块)
## 损失函数-Loss Function
损失函数（loss function）是用来衡量模型输出与真实标签之间的差异，当模型输出越接近标签，认为模型越好，反之亦然。因此，可以得到一个近乎等价的概念，loss越小，模型越好。这样就可以用数值优化的方法不断的让loss变小，即模型的训练。

针对不同的任务有不同的损失函数，例如回归任务常用MSE(Mean Square Error)，分类任务常用CE（Cross Entropy），这是根据标签的特征来决定的。而不同的任务还可以对基础损失函数进行各式各样的改进，如Focal Loss针对困难样本的设计，GIoU新增相交尺度的衡量方式，DIoU新增重叠面积与中心点距离衡量等等。

```python
nn.L1Loss
nn.MSELoss
nn.CrossEntropyLoss
nn.CTCLoss
nn.NLLLoss
nn.PoissonNLLLoss
nn.GaussianNLLLoss
nn.KLDivLoss
nn.BCELoss
nn.BCEWithLogitsLoss
nn.MarginRankingLoss
nn.HingeEmbeddingLoss
nn.MultiLabelMarginLoss
nn.HuberLoss
nn.SmoothL1Loss
nn.SoftMarginLoss
nn.MultiLabelSoftMarginLoss
nn.CosineEmbeddingLoss
nn.MultiMarginLoss
nn.TripletMarginLoss
nn.TripletMarginWithDistanceLoss
```
### 常见的损失函数及其优缺点
#### 均方误差 (Mean Squared Error, MSE)

 - 描述：计算预测值与真实值之间的平方差的均值。 
 - 适用场景：回归问题。 
 - 优点：便于梯度下降，误差大时下降快，误差小时下降慢，有利于函数收敛。
 - 缺点：受明显偏离正常范围的离群样本的影响较大。

#### 平均绝对误差 (Mean Absolute Error, MAE)

 - 描述：计算预测值与真实值之间的绝对差的均值。 
 - 适用场景：回归问题，想格外增强对离群样本的健壮性时使用。 
 - 优点：其克服了 MSE 的缺点，受偏离正常范围的离群样本影响较小。
 -  缺点：收敛速度比 MSE 慢，因为当误差大或小时其都保持同等速度下降，而且在某一点处还不可导，计算机求导比较困难。

#### 交叉熵损失 (Cross Entropy Loss)

 - 描述：衡量两个概率分布之间的差异。 
 - 适用场景：分类问题，特别是多分类问题。 
 - 优点：能够很好地反映分类的不确定性，对于分类问题非常有效。
 - 缺点：对于某些异常值或噪声敏感，特别是在二分类问题中，如果真实标签与预测概率相差很大，则损失值会非常高。

#### 二元交叉熵损失 (Binary Cross Entropy Loss)

 - 描述：交叉熵损失的特殊情况，用于二分类问题。 
 - 适用场景：二分类问题。 
 - 优点：与交叉熵损失类似，能够很好地反映分类的不确定性。
 - 缺点：同样对于异常值或噪声敏感。

#### 铰链损失 (Hinge Loss)

 - 描述：用于支持向量机（SVM）等模型，旨在最大化分类间隔。 
 - 适用场景：二分类问题。
 - 优点：能够最大化分类间隔，对于线性可分的数据集效果很好。 
 - 缺点：对于非线性可分的数据集效果不佳，且不直接考虑概率估计。

#### 0-1 损失 (0-1 Loss)

 - 描述：对每个错分类点都施以相同的惩罚。 
 - 适用场景：分类问题。 
 - 优点：直观简单，易于理解。
 -  缺点：不是连续可微的，因此不适合用于梯度下降等优化算法。

#### 平滑 L1 损失 (Smooth L1 Loss)

 - 描述：结合了 MSE 和 MAE 的优点，对于较大的误差使用 MSE，对于较小的误差使用 MAE。 
 - 适用场景：回归问题。 
 - 优点：兼具 MSE 和 MAE 的优点，对异常值较不敏感。 
 - 缺点：相对于 MSE 或 MAE 更复杂一些。

### 优化器-Optimizer
有了数据、模型和损失函数，就要选择一个合适的优化器(Optimizer)来优化该模型，使loss不断降低，直到模型收敛。优化器是根据权重的梯度作为指导，定义权重更新的力度，对权重进行更新。

**梯度哪里来？** 梯度通过loss的反向传播，得到每个权重的梯度值，其中利用pytorch的autograd机制自动求导获得各权重的梯度。 

**更新哪些权重？** 通过loss的反向传播，模型(nn.Module)的权重（Parameter）上有了梯度(.grad)值，但是优化器对哪些权重进行操作呢？实际上优化器会对需要操作的权重进行管理，只有被管理的权重，优化器才会对其进行操作。在Optimizer基类中就定义了add_param_group()函数来实现参数的管理。通常在实例化的时候，第一个参数就是需要被管理的参数。

**怎么执行权重更新？** step()函数是进行优化操作的，step()函数中实现了对所管理的参数进行更新的步骤。
#### 优化器基类 Optimizer
**Optimizer 基类的主要属性**
**defaults：**

 - 描述：一个字典，包含优化器的默认超参数值。 
 - 用途：在构造函数中初始化，用于设置优化器的默认配置。

**state：**

 - 描述：一个字典，用于存储有关参数的状态信息，如动量缓存等。 
 - 用途：在每次更新参数时，根据状态信息调整参数的更新方式。

**param_groups：**

 - 描述：一个列表，包含参数组的字典。 
 - 用途：允许对不同参数组使用不同的超参数，例如不同的学习率或正则化系数。

**step_count：**

 - 描述：一个计数器，记录优化器的更新次数。 
 - 用途：常用于学习率调度器，以基于迭代次数调整学习率。

**Optimizer 基类的主要方法**
**\__init__(params, defaults)：**

 - 描述：构造函数，初始化优化器。

 - 参数：

	 - params：一个迭代器，包含要优化的张量列表或者包含张量列表的字典列表。

	 - defaults：一个字典，包含默认的超参数值。

**zero_grad(set_to_none=False)：**

 - 描述：清空所有参数的梯度缓存。
 - 参数：
	 - set_to_none：布尔值，默认为 False。如果为 True，则将梯度设置为 None 而不是清零。

**step(closure=None)：**

 - 描述：执行单步优化。
 - 参数：
	 - closure：一个可选的无参函数，用于重新评估模型损失并返回它。此函数通常用于计算梯度前的损失值，以便在下一步更新参数时使用。

**add_param_group(param_group)：**

 - 描述：添加一个新的参数组到优化器中。
 - 参数：
	 - param_group：一个字典，包含新的参数组。

**load_state_dict(state_dict)：**

 - 描述：从状态字典加载优化器的状态。
 - 参数：
	 - state_dict：一个字典，包含优化器的状态。

**state_dict()：**

 - 描述：返回一个包含优化器当前状态的字典。
 - 返回：一个字典，包含优化器的状态信息。

#### 常见的优化器及其优缺点
**随机梯度下降 (Stochastic Gradient Descent, SGD)**

 - 描述：是最基本的优化算法之一，使用损失函数关于模型参数的梯度来更新参数。 
 - 优点：简单易懂，计算效率高。
 - 缺点：容易陷入局部最小值，学习率难以选择。

**动量优化器 (Momentum)**

 - 描述：在 SGD 的基础上引入动量项，使得更新过程更加平稳。 
 - 优点：有助于加速收敛过程，减少振荡。 
 - 缺点：需要额外的参数来控制动量的大小。

**AdaGrad**

 - 描述：自适应学习率的方法，根据历史梯度的平方和来调整每个参数的学习率。 
 - 优点：能够自动调整学习率，有助于解决稀疏数据的问题。
 - 缺点：学习率随时间单调递减，可能会过早停止学习。

**RMSprop**

 - 描述：改进版的 AdaGrad，通过指数加权平均来计算历史梯度的平方和。 
 - 优点：解决了 AdaGrad 学习率过早衰减的问题。
 - 缺点：需要手动设置一些超参数。

**Adam (Adaptive Moment Estimation)**

 - 描述：结合了 Momentum 和 RMSprop 的优点，同时计算梯度的一阶矩估计和二阶矩估计。
 - 优点：计算效率高，通常表现良好，不需要手动调整学习率。 
 - 缺点：在某些情况下可能收敛到次优解。

**Adadelta**

 - 描述：类似于 RMSprop，但不需要手动设置学习率。 
 - 优点：不需要手动设置学习率，减少了调参的难度。
 - 缺点：在某些情况下可能收敛速度较慢。

**AdamW**

 - 描述：Adam 的改进版本，解决了 Adam 在权重衰减方面的不足。 
 - 优点：更好地处理了权重衰减的问题，有助于提高模型的泛化能力。
 - 缺点：需要更多的计算资源。

**torch.optim.Adam** 是 PyTorch 提供的一种优化器，实现了 Adam 优化算法。Adam（Adaptive Moment Estimation）是一种自适应学习率的方法，它结合了 AdaGrad 和 RMSProp 的优点，通过维护第一矩估计（均值）和第二矩估计（未中心化的方差）来动态调整每个参数的学习率。
Adam 优化器参数

 - params：待优化的参数，通常传入的是模型的参数 model.parameters()。 
 - lr (float, optional)：学习率（默认：1e-3）。这是每个参数更新的基础学习率。 
 - betas (Tuple[float, float], optional)：系数用于运行平均值和运行平方值的计算（默认：(0.9, 0.999))。betas[0] 对应于第一矩估计的衰减率，betas[1] 对应于第二矩估计的衰减率。
 -  eps (float,  optional)：增加到分母中的值，以提高数值稳定性（默认：1e-8）。 
 - weight_decay (float, optional)：权重衰减（L2 正则化）（默认：0）。 
 - amsgrad (bool, optional)：是否使用 AMSGrad 版本的 Adam 算法（默认：False）。AMSGrad 是 Adam 的一个变种，它通过维护最大历史第二矩来解决 Adam 收敛速度可能慢于预期的问题。

### 学习率调整策略
学习率调整策略（Learning Rate Scheduling）是在训练过程中动态调整学习率的方法，这对于提高模型的收敛速度和最终性能非常重要。适当的学习率调整可以帮助模型更好地探索损失函数的曲面，从而找到更好的局部最小值或全局最小值。
#### 核心属性
**optimizer：**

 - 描述：这是被调度的优化器对象。
 - 作用：学习率调度器需要知道它正在调整哪个优化器的学习率。
 - 示例：`optimizer = optim.SGD(model.parameters(), lr=0.01)`
**base_lrs：**
 - 描述：这是优化器中各组参数的初始学习率列表。 
 - 作用：学习率调度器需要知道每个参数组的初始学习率，以便能够正确地调整它们。
 - 示例：如果模型中有两个参数组，那么 base_lrs 将是一个包含两个学习率的列表。 
 - 获取方式：`self.base_lrs =  [group['initial_lr'] for group in optimizer.param_groups]`

**last_epoch：**

 - 描述：这是最后一次调用 scheduler.step() 时的训练周期数。
 - 作用：学习率调度器需要跟踪当前的训练周期数，以便能够根据周期数调整学习率。 
 - 默认值：-1 表示尚未调用 scheduler.step()。
 -  示例：`scheduler = MyCustomScheduler(optimizer, last_epoch=-1)`

#### 核心方法
**state_dict()：**

 - 描述：返回一个包含学习率调度器状态的数据字典。 
 - 作用：可以用来保存学习率调度器的状态，以便后续恢复。

**load_state_dict(state_dict)：**

 - 描述：从给定的状态字典中加载学习率调度器的状态。 
 - 作用：用于恢复之前保存的学习率调度器状态。

**get_last_lr()：**

 - 描述：返回上一次调用 scheduler.step() 之后的学习率列表。 
 - 作用：可以用来获取当前的学习率。

**get_lr()：**

 - 描述：计算当前的学习率列表。 
 - 作用：在 scheduler.step() 调用之前计算新的学习率。

**print_lr()：**

 - 描述：打印当前的学习率。
 -  作用：方便调试和监控学习率的变化。

**step(epoch=None)：**

 - 描述：更新每组参数的学习率。 
 - 作用：在每个训练周期结束时调用，以更新学习率。
 - 参数：epoch（可选）表示当前的训练周期数，如果不提供，则使用 last_epoch + 1。


#### lr_scheduler 使用流程
在 PyTorch 中使用学习率调度器 (lr_scheduler) 的基本流程包括以下几个步骤：

 1. 创建模型：首先定义并实例化您的模型。 
 2. 定义损失函数：选择适合您任务的损失函数。 
 3. 创建优化器：根据您的需求选择合适的优化算法，如SGD、Adam 等，并设置初始学习率。 
 4. 创建学习率调度器：选择合适的学习率调度策略，并使用优化器实例创建调度器。
 5.  训练模型：在训练循环中使用优化器更新模型参数，并在适当的时机调用调度器来更新学习率。
 6. 监控学习率：在训练过程中监控学习率的变化，以确保调度策略按预期工作。

#### PyTorch 中常用的学习率调整器（学习率调度器）的汇总及其特点
1. StepLR
特点：
定期将学习率乘以一个给定的因子。
适用于需要定期降低学习率的情况。
参数：
optimizer：要调整的学习率的优化器。
step_size：学习率调整的周期。
gamma：学习率调整的比例因子，默认为 0.1。
2. MultiStepLR
特点：
在多个指定的周期点将学习率乘以一个给定的因子。
更灵活地控制学习率下降的时间点。
参数：
optimizer：要调整的学习率的优化器。
milestones：一系列学习率调整的时间点。
gamma：学习率调整的比例因子，默认为 0.1。
3. ExponentialLR
特点：
按照指数衰减的方式调整学习率。
适用于需要平滑地降低学习率的情况。
参数：
optimizer：要调整的学习率的优化器。
gamma：学习率调整的比例因子。
4. CosineAnnealingLR
特点：
学习率按照余弦函数的方式衰减至最小值。
适用于需要平滑地降低学习率至某个最小值的情况。
参数：
optimizer：要调整的学习率的优化器。
T_max：学习率衰减周期的最大长度。
eta_min：学习率的最小值，默认为 0。
5. ReduceLROnPlateau
特点：
根据验证集上的性能（如损失或准确率）来动态调整学习率。
适用于需要根据模型性能来调整学习率的情况。
参数：
optimizer：要调整的学习率的优化器。
mode：监测指标的优化方向（min 或 max）。
factor：学习率调整的比例因子。
patience：在多少个周期内未见改进前调整学习率。
6. CyclicLR
特点：
在一定周期内循环地调整学习率。
适用于需要周期性地提高和降低学习率的情况。
参数：
optimizer：要调整的学习率的优化器。
base_lr：周期内的最低学习率。
max_lr：周期内的最高学习率。
step_size_up：学习率从 base_lr 到 max_lr 的周期长度。
step_size_down：可选，学习率从 max_lr 回降到 base_lr 的周期长度。
7. OneCycleLR
特点：
在一个周期内将学习率从高到低再到高，最后再回到低。
适用于需要快速找到最优学习率范围的情况。
参数：
optimizer：要调整的学习率的优化器。
max_lr：周期内的最高学习率。
total_steps：总的训练步数。
pct_start：学习率从高到低的周期占总周期的比例。
8. LambdaLR
特点：
根据用户提供的 lambda 函数来调整学习率。
适用于需要自定义学习率调整策略的情况。
参数：
optimizer：要调整的学习率的优化器。
lr_lambda：一个函数或函数列表，用于计算每个周期的学习率。
9. CosineAnnealingWarmRestarts
特点：
学习率在每个重启周期内按照余弦函数的方式衰减至最小值，然后在下一个周期重新开始。
适用于需要周期性地调整学习率的情况。
参数：
optimizer：要调整的学习率的优化器。
T_0：第一个重启周期的长度。
T_mult：后续重启周期长度的增长因子。
eta_min：学习率的最小值，默认为 0。


参考：[https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-5/](https://tingsongyu.github.io/PyTorch-Tutorial-2nd/chapter-5/)
参考：[https://pytorch-cn.readthedocs.io/zh/latest/](https://pytorch-cn.readthedocs.io/zh/latest/)
参考：[https://datawhalechina.github.io/thorough-pytorch/](https://datawhalechina.github.io/thorough-pytorch/)

