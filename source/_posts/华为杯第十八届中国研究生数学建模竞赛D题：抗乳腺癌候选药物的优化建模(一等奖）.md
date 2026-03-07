---
title: 华为杯第十八届中国研究生数学建模竞赛D题：抗乳腺癌候选药物的优化建模(一等奖）
date: 2026-01-04 08:03:00
tags:
  - "数学建模"
  - "研究生竞赛"
  - "华为杯"
  - "一等奖"
categories:
  - "数学建模"
---

﻿**更新20220921：参加数模之旅需要哪些准备？（转自中国研究生数学建模竞赛公众号）**
●  前期知识储备
公众号、博客、知乎、纸质书籍等
●  熟悉题型
华为题(A题)：与电子信息专业相关度高
大数据类：神经网络、Python为主
传统优化类：gurobi工具、整数规划、遗传算法、蚁群算法
●  技能学习
matlab、C++、python、R等科学计算语言
机器学习
数据分析
数据可视化
多目标决策
线性规划
●  软件学习
论文编写：Word、LaTeX (量力而行) 
公式编写：Mathtype
绘图软件：Visio、亿图
数据可视化：Excel、SPSS
**更新  2022/04/24**
听了一等奖的学习交流会议，发现一等奖的思路都是差不多的，有一些有生物化学方面基础的可能对这方面的补充更多一些，计算机专业的对模型方面补充更多一些。有一组数模提名的队伍把答辩ppt和论文都分享了，受益匪浅。以下是github地址：
[https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)

[微信公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487303&idx=1&sn=a2bdb7260d6508655e5da4366817744a&scene=19#wechat_redirect)
@[TOC](抗乳腺癌候选药物的优化建模)
# 摘       要
乳腺癌是目前世界上最常见，致死率较高的癌症之一。在寻找抗乳腺癌候选药物时，需同时保证化合物的生物活性和药代动力学性质和安全性。本文采用数据挖掘技术，研究了抗乳腺癌候选药物的优化建模问题。
**针对问题一**，首先对所有化合物的分子描述符进行数据处理，进行缺失值和重复值检查，经检查，未发现缺失值和重复值。假设附件一和附件二的数据是对分子化合物的真实情况记录，未对异常值进行处理。之后剔除了分子描述符所在列是唯一值的列。经处理后，描述符数量由729个减少到504个。考虑到变量的取值特征，本文将504个分子描述符变量分为连续型变量（物理化学性质）和离散型变量（拓扑结构特征）两部分，分别运用相关分析和方差选择的方法，选择了与生物活性存在相关关系较强的100个连续型和55个离散型分子描述符。对这155个自变量与PIC50值建立LightGBM回归模型，并且对自变量的贡献度进行排序，找到前23个显著影响化合物生物活性的因素。考虑到这23个自变量之间可能存在多重共线性，为保证变量有较高的解释程度，计算自变量之间的相关系数，剔除自变量之间相关性较高的变量，最终得到对生物活性最具显著影响的20个分子描述符变量。最后对选取的变量计算MIC和Spearman值，结果表明，选取的变量之间相关关系较弱，具有很好的独立性。同时，选取的20个变量在化学意义上具有很好的可解释性，说明20个变量的选取是合理的。
**针对问题二**，本文选择了两种模型进行对比，分别采用了随机森林和LightGBM回归模型。选取问题一得到的20个分子描述符变量，首先采用KDE分布图对比了训练集和测试集中特征变量的分布情况，剔除了数据集中分布不一致的特征变量。考虑到数据的离散性和连续性，以及自变量和因变量之间可能存在非线性关系，而且数据集较小，容易过拟合，所以本文选择随机森林和LightGBM做回归，并结合了K折交叉检验法。最后对比两组模型的误差评价指标MAE、MSE和拟合系数R2，结果显示LightGBM模型要优于随机森林方法，最终选取LightGBM方法对化合物IC50值和pIC50值进行定量预测。
**针对问题三**，选取问题一得到的20个分子描述符变量对化合物的ADMET性质构建分类预测模型，本文使用了DNN和LightGBM分类模型。使用DNN模型对标准化的数据做分类预测，LightGBM使用原数据做分类预测，最后对两个模型的结果求平均，最终得到预测结果。其中DNN网络选用Sigmoid激活函数，使用优化算法Adam加快收敛速度。为了防止过拟合，使用dropout方法对数据进行训练。考虑到样本不平衡问题，使用LightGBM模型中的subsample参数进行处理。对ADMET测试集进行预测，得到预测结果。模型评价指标选取AUC指标，五个分类模型的值均在0.9以上，说明模型拟合较好。
**针对问题四**，结合问题一、二、三，选取问题一得到的对生物活性具有显著影响的20个分子描述符特征变量，使用问题二的回归模型和问题三的分类模型，结合粒子群优化算法，进行问题求解。首先，为保证化合物的生物活性，以IC50最小（即pIC50最大）为目标函数。同时需要对ADMET性质进行约束，以保证至少三个较好的性质为约束条件。通过基于LightGBM模型的粒子群优化算法，对特征变量的取值范围进行搜索优化，最终获得相应取值范围。

**关键词：分子描述符，特征选择，LightGBM模型，DNN模型，粒子群优化算法**
# 1. 问题重述
## 1.1 问题背景
乳腺癌是目前世界上最常见，致死率较高的癌症之一。ERα被认为是治疗乳腺癌的重要靶标，能够拮抗ERα活性的化合物可能是治疗乳腺癌的候选药物。比如，临床治疗乳腺癌的经典药物他莫昔芬和雷诺昔芬就是ERα拮抗剂。
在药物研发中，为了节约时间和成本，通常采用建立化合物活性预测模型的方法来筛选潜在活性化合物。以一系列分子结构描述符作为自变量，化合物的生物活性值作为因变量，构建化合物的定量结构-活性关系（Quantitative Structure-Activity Relationship, QSAR）模型，然后使用该模型预测具有更好生物活性的新化合物分子，或者指导已有活性化合物的结构优化。一个化合物想要成为候选药物，除了需要具备良好的生物活性（此处指抗乳腺癌活性）外，还需要在人体内具备良好的药代动力学性质和安全性，合称为ADMET（Absorption吸收、Distribution分布、Metabolism代谢、Excretion排泄、Toxicity毒性）性质。
根据提供的ERα拮抗剂信息（1974个化合物样本，每个样本都有729个分子描述符变量，1个生物活性数据，5个ADMET性质数据），构建化合物生物活性的定量预测模型和ADMET性质的分类预测模型，进而为优化ERα拮抗剂的生物活性和ADMET性质服务。
## 1.2 问题重述
基于上述研究背景，本文需研究和解决以下问题：
**问题一 筛选分子描述符**
根据文件“Molecular_Descriptor.xlsx”和“ERα_activity.xlsx”提供的数据，针对1974个化合物的729个分子描述符进行变量选择，根据变量对生物活性影响的重要性进行排序，并给出前20个对生物活性最具有显著影响的分子描述符（即变量），并请详细说明分子描述符筛选过程及其合理性。
**问题二 生物活性定量预测**
在问题一的基础上，选择不超过20个分子描述符变量，构建化合物对ERα生物活性的定量预测模型。使用构建的预测模型，对文件“ERα_activity.xlsx”的test表中的50个化合物进行IC50值和对应的pIC50值预测。
**问题三 ADMET性质分类预测**
利用文件“Molecular_Descriptor.xlsx”提供的729个分子描述符，针对文件“ADMET.xlsx”中提供的1974个化合物的ADMET数据，分别构建化合物的Caco-2、CYP3A4、hERG、HOB、MN的分类预测模型。然后使用所构建的5个分类预测模型，对文件“ADMET.xlsx”的test表中的50个化合物进行相应的预测。
**问题四 寻找分子描述符取值范围**
寻找并阐述化合物的哪些分子描述符，以及这些分子描述符在什么取值或者处于什么取值范围时，能够使化合物对抑制ERα具有更好的生物活性，同时具有更好的ADMET性质（给定的五个ADMET性质中，至少三个性质较好）。
# 2. 模型假设
假设 1：所有样本的数据记录均为化合物的真实值、不存在录入误差，数据处理步骤正确；
假设 2：影响抗乳腺癌候选药物生物活性的因素只与729个分子描述符有关； 
假设 3：在寻找分子描述符范围时认为所提出的预测模型结果准确。
# 3. 符号说明
本文涉及符号较多，因此选择了一部分重要符号列出在下表。其他符号在文中均有说明。
![在这里插入图片描述](/img/00870757f3e0342b759af9c526d9dc5c.png)


# 4. 问题一 筛选最具显著影响描述符
## 4.1 问题分析
 根据文件“Molecular_Descriptor.xlsx”和“ERα_activity.xlsx”提供的数据，针对1974个化合物的729个分子描述符进行变量选择。根据附件“分子描述符含义解释.xlsx”的解释，可以看出分子描述符被分为54类，变量之间是存在相关性或独立性的。本题的思路流程如图所示： 
![图 1问题一思路流程图](/img/ff676199bbe748e13046f0f9cc0f7360.png)
针对729个分子描述符，本文首先希望对其进行降维操作，剔除最不相关的变量，挑选出一部分具有代表性和独立性性的变量。本文的难点在于：（1）各自变量（生物活性）和因变量（分子描述符）之间具有高度非线性关系，判定因、自变量相关程度较为困难。而且，分子描述符中包括了物理化学性质（如分子量，LogP等），拓扑结构特征（如氢键供体数量，氢键受体数量等），本文认为不能同时对这些变量进行操作。同时为了问题二和问题四的解决，选择的变量必须是原有变量，这是特征选取问题，无法使用较为常规的特征提取方法；（2）由于变量过多，变量与变量之间可能存在相互强耦连的关系，故选取变量的独立性问题较难处理。
针对难点（1），变量的选择问题，筛选具有代表性的变量。首先，筛除变量中最具一般性的描述符。分子描述符分为组成描述符、分子性质描述符、拓扑描述符、几何描述符等，本文认为可以将其分为两类分别处理。本文将自变量分为连续变量和离散变量，分别对其进行初步选择。特征选择为从给定的特征中直接选择若干重要特征，所选取的变量必须是客观的，非负矩阵分析、主成分分析、独立成分分析等不适用于此问题。故最后采用LightGBM算法获取到各变量对生物活性贡献度的排名，依此实现对选取变量代表性的判断。
针对难点（2），变量的独立性问题，根据LightGBM得到分子描述符的贡献度排名后，对前25个变量进行多重共线性处理，从高度相关的自变量中进行筛除，来保证最后变量间的独立性。
最后，本文对得到的最具显著影响的变量进行合理性评价。
## 4.2 变量初步筛选
### 4.2.1 数据处理
对附件“Molecular_Descriptor.xlsx”中的数据进行缺失值检查，未发现数据缺失。针对异常值问题，本文选择不对异常值处理，考虑到所给数据是记录的化合物分子描述符的真实值，直接在此基础上进行数据挖掘能够保留最真是可信的信息。
### 4.2.2 变量的初步筛选
首先，本文对729列分子描述数据进行了唯一值检查，并剔除了所在列为唯一值的变量。本文认为所在列为唯一值的分子描述对于化合物是一般性质，不具有代表性，所以进行了剔除。经过唯一值检查后，变量由729个缩减为504个。考虑到变量的连续性和离散性，本文对504个变量进行了分类，分别进行处理。下图绘制了IC50的直方图和QQ图，验证其近似服从正态分布：
![图 2 pIC50的直方图和QQ-图](/img/3b9658dca8188768c40fb77af4cc3ecc.png)
本文认为在样本集上如果当前特征基本上都差异不大，因此可以认为这个特征对区分样本贡献不大，因此可以在选择特征过程中可以将其去掉。针对离散型变量，采用了方差选择方法，从中选择了50个变量。对于连续型变量，采用相关分析的方法，选择了100个连续变量。最初步筛选后得到150个变量。
**（1）方差选择法**
方差法，要先计算各个特征的方差，然后根据阈值，选择方差大于阈值的特征。一组常数的方差为0，数据的变化越小，则方差越小。设定方差阀值，若特征的方差小于阈值，则代表该特征的发散性太弱，对于因变量几乎没有影响，可以舍弃。
**（2）Pearson相关系数法**
对特征变量的相关性进行分析，可以发现特征变量和目标变量及特征变量之间的关系，计算100个连续自变量和因变量之间的相关性系数。两个变量之间的皮尔逊相关系数定义为两个变量之间的协方差和标准差的商：

## 4.3 LightGBM算法——变量重要性排序	9
## 4.4 多重线性分析	10
## 4.5 合理性解释	12
# 5. 问题二 生物活性定量预测	12
## 5.1 问题分析	13
## 5.2 生物活性定量预测模型建立	14
### 5.2.1 基于KDE分布图剔除特征变量	14
### 5.2.2 K折交叉验证法	15
### 5.2.3 随机森林算法实现	16
### 5.2.4 基于LightGBM的回归模型	17
### 5.2.5 模型比较	18
## 5.3 预测结果与分析	19
# 6. 问题三 ADMET性质分类预测	21
## 6.1 问题分析	21
## 6.2 数据处理	22
### 6.2.1 一般性检验	22
### 6.2.2 数据标准化	22
## 6.3 ADMET性质分类预测模型建立	23
### 6.3.1 DNN基本原理	23
### 6.3.2 DNN模型设计	24
### 6.3.3 基于LightGBM的分类模型	27
### 6.3.4 ADMET性质分类模型的建立	28
## 6.4 分类结果与分析	28
# 7. 问题四 分子描述符寻找及取值范围	32
## 7.1 问题分析	32
## 7.2 选择分子描述符的优化模型建立	33
### 7.2.1 粒子群算法	33
### 7.2.2 优化目标及条件设定	35
### 7.2.3 模型参数设定	36
## 7.3 结果与分析	37
# 8.模型的评价与改进	38
# 9. 参考文献	39

```python
问题一 python程序	变量筛选
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats
from sklearn.feature_selection import VarianceThreshold

#%%

ADMET_training=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\ADMET.xlsx',sheet_name='training')
ADMET_test=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\ADMET.xlsx',sheet_name='test')
ADMET_training.head()
#ADMET_test.head()

#%%

ER_activity_training=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\ERα_activity.xlsx',sheet_name='training')
ER_activity_test=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\ERα_activity.xlsx',sheet_name='test')
ER_activity_training.head()
#ER_activity_test.head()

#%%

Molecular_Descriptor_training=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\Molecular_Descriptor.xlsx',sheet_name='training')
Molecular_Descriptor_test=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\Molecular_Descriptor.xlsx',sheet_name='test')
Molecular_Descriptor_training.head()
#Molecular_Descriptor_test.head()

#%%

Summary=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\分子描述符含义解释.xlsx',sheet_name='Summary')
Detailed=pd.read_excel(r'C:\Users\Administrator\project\huaweibeiD\分子描述符含义解释.xlsx',sheet_name='Detailed')

#%%

for col in Molecular_Descriptor_training.columns:
    #nunique() 方法用于获取某列中所有唯一值的数量，
    #dropna 默认参数设置为True，即在计算唯一值时排除了NULL值。    
    if Molecular_Descriptor_training[col].nunique(dropna=False)==1:
        del Molecular_Descriptor_training[col]
    # 去掉只有一种类别的 columns
len(Molecular_Descriptor_training.columns)
#729->504

#%%

True in Molecular_Descriptor_training.isna().sum()!=0
#False：数据没有缺失值

#%%

Molecular_ER = pd.concat([Molecular_Descriptor_training, ER_activity_training[:]], axis=1)
del Molecular_ER['SMILES']
del Molecular_ER['IC50_nM']
Molecular_ER

#%%

#pIC50直方图和QQ图
plt.figure(figsize=(10,5),dpi=400)
ax=plt.subplot(1,2,1)
sns.distplot(Molecular_ER['pIC50'],fit=stats.norm)
ax=plt.subplot(1,2,2)
res=stats.probplot(Molecular_ER['pIC50'],plot=plt)
plt.savefig('pic50_QQ.png')

#%%

#离散特征
Discrete_features=[]
for i in Detailed['Descriptor']:
    if i[0]=='n' and i in Molecular_Descriptor_training.columns:
        Discrete_features.append(i)
#Molecular_Discrete_training=Molecular_Descriptor_training[Discrete_features]
#连续特征
Continuous_features=[col for col in Molecular_Descriptor_training.columns if col not in Discrete_features+['SMILES']] 
#Molecular_Continuous_training=Molecular_Descriptor_training[Continuous_features]
print(len(Discrete_features),len(Continuous_features))
......
```

# （更新时间：2022/08/15）[2021年优秀论文已经公布](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487303&idx=1&sn=a2bdb7260d6508655e5da4366817744a&chksm=ec0c1d98db7b948e36e07e4ffc9b1a32f826204245eb60d3b5d5ffa24fd0aa651876f5462a88&token=440316933&lang=zh_CN#rd)

