---
title: CL 成绩
author: Jing
date: 2024-04-26
tags:
  - 比赛
  - 方向探索
---
无标签数据利用、发现新类（存在在 unlable 类中）、
模糊任务边界、剔除干扰类、
不使用预训练、不存储真实数据集（允许存储数据表示）、GPU memory 受限、模型受限
对比赛的理解：
- 无标签数据的利用：用来发现新类，同时巩固旧的记忆（知识蒸馏）
	- Continual Generalized Category Discovery：连续广义类发现
	- ==该赛题中不需要发现新类，只需要不影响旧的记忆即可==
- 模糊任务边界：增加一个头，来确定其是否属于之前的类别
- 存储数据表示：找不存储真实数据集的方法
- 模型优化：找一些轻量化的模块
- 学习率优化：如 CLIB 中的学习率设置
# 1. 比赛结果

|                     方法                      | average accurary | convergence rate | accurary s3 | accurary s2 | accurary s1 |
| :-----------------------------------------: | :--------------: | :--------------: | :---------: | :---------: | :---------: |
|                  baseline                   |      0.0954      |      0.007       |   0.1020    |   0.1054    |   0.0788    |
|             unlabel<br>将未来数据去除              |     0.03773      |                  |             |             |             |
|             consine_similarity              |      0.0582      |                  |             |             |             |
|             pearson_correlation             |     0.06393      |                  |             |             |             |
| suprivsed_contrastive_learning<br>one stage |     0.03893      |                  |             |             |             |
| suprivsed_contrastive_learning<br>two stage |                  |                  |             |             |             |
|                   clib_my                   |                  |                  |             |             |             |
unlabel 降的原因可能是因为单个模型效果就不是很好，所以效果变差
# 2. 计划

| 方法       | 代码                                                                                                                                                                                                                            | 论文                                                                                                                                         | 说明  |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | --- |
| LWF      | [ContinualAI/clvision-challenge-2024: 5th CLVISION workshop at CVPR: repo for the challenge (github.com)](https://github.com/ContinualAI/clvision-challenge-2024)                                                             | [Learning without Forgetting](https://arxiv.org/pdf/1606.09282)                                                                            |     |
| MEMO     | [wangkiw/ICLR23-MEMO：PyTorch 中“A Model or 603 Exemplars： Towards Memory-Efficient Class-Incremental Learning”（ICLR'23）的代码存储库 (github.com)](https://github.com/wangkiw/ICLR23-MEMO)                                            | [[2205.13218] A Model or 603 Exemplars: Towards Memory-Efficient Class-Incremental Learning (arxiv.org)](https://arxiv.org/pdf/2205.13218) |     |
| MVP      | [KHU-AGI/Si-Blurry: Official repository for Online Class Incremental Learning on Stochastic Blurry Task Boundary via Mask and Visual Prompt Tuning on ICCV 2023 (github.com)](https://github.com/KHU-AGI/Si-Blurry/tree/main) | [2308.09303 (arxiv.org)](https://arxiv.org/pdf/2308.09303)                                                                                 |     |
| CLIB     |                                                                                                                                                                                                                               |                                                                                                                                            |     |
| DP       |                                                                                                                                                                                                                               |                                                                                                                                            |     |
| gdumb    |                                                                                                                                                                                                                               |                                                                                                                                            |     |
| iNCD     |                                                                                                                                                                                                                               |                                                                                                                                            |     |
| Meta-GCD |                                                                                                                                                                                                                               |                                                                                                                                            |     |
# 3. 策略详解
> 本次比赛以前观察到的类可以以任意重复模式重新出现在新体验中，并且并非所有课程在每种体验中都可用。
> “任务干扰”（task interference）或“知识干扰”（knowledge interference）。
> ==task-free continual learning==、==blurry task incremental==
> 意味着数据分布是动态变化的。
## 3.1 ~~MEMO~~
- 适用方向：不同 stream 之间的标签不相重叠
其中每个 experience 中都有一个单独的头
## 3.2 ~~MVP~~
- 使用了预训练
## 3.3 

# 4. 遇到的问题
- 如何利用未标记的数据？
- 如何学习这种可能重复的模式？
	- 这种模式叫 blurry task incremental
- 如何应对这种干扰类别？
- blurry class 和 blurry task
	- blurry classes 的意思是类与类之间的边界不清晰或存在重叠的情况。
	-  blurry task 的意思是任务与任务之间的边界不清晰或存在重叠的情况
- stochastic incremental blurry task boundary scenario 
# 5. 论文
## 5.1 类增量学习
### 5.1.1 `Online Class Incremental Learning on Stochastic Blurry Task boundary via Mask and Visual Prompt Tuning`
ICCV 2023
提出了 stochastic incremental blurry task boundary scenario，叫做 Si-Blurry，可以反映现实世界中的随机特性。
在 Si-Blurry 场景下，有两个主要性能下降的原因：（1）任务内和任务间遗忘；（2）类别不平衡问题。
持续学习的一种划分方式：disjoint continual learning、blurry continual learning
[3] Rainbow memory: Continual learning with a memory of diverse samples. 提出 online learning with blurry task boundaries
[18] Online continual learning on class incremental blurry task configuration with anytime inference. 提出 i-Blurry scenario，结合不相交的持续学习（disjoint continual learning）和模糊的无任务持续学习（blurry task-free continual learning）
 i-Blurry 任务之间的类数量是固定的。
 ==blurry continual learning 是一个有趣的方向==
 ==class overlap==
> 使用预训练模型的知识进行持续学习

MVP 方法：
- 提出了 instance-wise logit masking 和 contrastive visual prompt tuning loss，来解决 intra 和 inter-task forgetting 问题
- 提出了 gradient similarity-based focal loss 和 adaptive feature scaling 去缓解类别不平衡问题
### 5.1.2 Continual Learning with Evolving Class Ontologies
[LECO：不断发展的班级本体的持续学习 (linzhiqiu.github.io)](https://linzhiqiu.github.io/papers/leco/)
> 人类先认识“狗”，再认识不同的犬种

### 5.1.3 Task-Free Continual Learning

### 5.1.4 continual learning and catastrophic forgetting
分类：
- task-based and task-free continual learning
- task-, domain- and class-incremental learning
stream-based continual learning and online continual learning
深度神经网络持续学习的六种主要的计算方法：
（i）重放，（ii）参数正则化，（iii）函数正则化，（iv）基于优化的方法，（v）上下文相关处理和（vi）模板为基础的分类。
 (i) replay, (ii) parameter regularization, (iii) functional regularization, (iv) optimization-based approaches, (v) context-dependent processing and (vi) template-based classification.
### 5.1.5 online continual learning on class incremental blurry task configuration with anytime inference

流式持续学习、在线持续学习
提出的持续学习设置式：online、task-free、class-incremental、blurry task boundaries 和 subject to inference queries at any moment
[Rahaf Aljundi NeurIPS 2019c]Gradient based sample selection for online continual learning. 
作者提出解决 i-Blurry 设置的方法：==Continual Learning for i-Blurry or CLIB==
- 设置了新的内存管理方案，使用每个样本的重要性得分来丢弃样本
	- 用来选择最优移除样本![[i_blurry_clib.png]]
	- ==是否可以照样子存数据表示，之后移除数据表示==
- 仅从内存中提取训练样本，而不是像 ER 中那样的内存和在线流中提取训练样本
- 学习率调度，根据损失轨迹自适应地决定增加和减少学习率（即数据驱动的方式）
	- 之前的方案：当遇到新类时，带有重置的指数调度程序将 LR 重置为初始值
	- 自适应 lr ![[i_blurry_clib_lr.png]]
	- ==这个自适应的 lr 也可以借鉴==

## 5.2 新类发现
> 在未标记数据中发现新类
### 5.2.2 MetaGCD: Learning to Continually Learn in Generalized Category Discovery
 class-iNCD[44]
 ==C-GCD==[21, 39, 28, 27]
 解决 C-GCD 的初步尝试[16, 42]，只在上述部署阶段考虑 C-GCD

元学习是一种机器学习范式，其目标是在学习过程中优化学习算法本身，而不是学习特定的任务。其核心思想是让模型学会如何学习，这样就可以更快、更有效地适应新任务。通常用于小样本学习和零样本学习场景。
主要方法和技术：优化算法（梯度下降、自适应学习率算法等）、网络结构、元策略
#### 问题描述
C-GCD 的目标是让离线训练的模型不断从包含已知类和新类的未标记数据中发现和学习新的对象类。
用标记数据进行离线训练；用未标记数据（已知类或未知类）进行增量学习。
#### 解决方法
MetaGCD 方法
- 使用元学习框架(meta-learning framework)
- 元目标(meta-objective)被定义未围绕两个相互冲突的学习目标：发现新类而不忘记旧类
- 基于软邻域的对比网络(a soft neighborhood-based contrastive network)来区分不相关图像，同时吸引相关图像
需要 automatic class discovery mechanism
#### 相关工作
- Continual Generalized Category Discovery(C-GCD)：目标是不断发现新的类别，同时保持已知类别的性能
	- C-GCD 的两阶段：(1) 离线训练阶段，允许模型在具有预定义类别的大规模标记数据上进行训练；(2) 当模型部署时，它会不断遇到来自已知类和新类的未标记数据。在每个增量会话中，之前会话的数据都无法访问。
- meta-learning：
	- 元学习的方法可以分为：(1) model-based; (2) optimization-based; (3) metric-based; 
	- 典型的 meta-learning 利用双层优化(bi-level optimization)来训练适用于下游任务的模型，当前工作建立在 MAML 之上，它通过任务片段训练模型初始化，以便通过梯度更新快速适应。
	- 适应是以无监督的方式实现的，并且利用双层优化来结合两个相互冲突的学习目标：发现新类而不忘记旧类。
	- meta-learning：learn to learn
- contrastive learning:
	- 利用对比学习减少标签空间上的过度拟合，以提高下游任务的泛化能力
#### 具体方法
- 没有参数分类头的模型，更适合处理新类别
- 直接对特征空间进行聚类来执行新颖类发现，通过经典的 k-means 算法分配类标签
- 在 offline training 阶段，使用标记数据学习模型初始化
- 为了充分利用 offline learning 中的标记数据，开发了基于元学习的双层优化来模拟在线学习场景。
	- 结合无监督和有监督对比损失
- 对于未标记数据，用软邻域对比学习(soft neighborhood contrastive learning)