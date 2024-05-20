---
title: Learning to Imagine Diversify Memory for Incremental Learning using Unlabeled Data
author: Jing
date: 2024-05-06
tags:
  - 持续学习
---
> 利用未标记数据进行增量学习的多样化记忆
> ==无标签数据用于生成样本==
> 不是 blurry task 的任务
# 1. 领域问题
领域：类增量
问题：现有的方法受限于样本数量较少，不能携带足够的任务特异性知识，遗忘问题存在。
# 2. 创新点

# 3. 具体方法
two-stage trainng schedule
- 第一个任务结束时，从当前数据集中抽取少量样本
- 删除原始数据前，训练 feature generator，基于样例和海量未标记数据生成多样的样例对应物
- 将生成的样本与原数据集进行==语义对比学习==( semantic contrastive learning)
	- 使生成的样本和示例在语义上保持一致
	- 鼓励生成的样本尽可能多样化
- 生成样本与未标记数据之间的==语义解耦对比学习==( semantic-decoupling contrastive learning)
	- 进一步促进对未标记数据中语义无关信息的探索并生成更多样的样本
- 新任务开始时，冻结特征生成器并用于生成多样化的样本
我们的方法在训练阶段不需要进一步微调，也不用带来额外的推理成本。
## 3.1 问题描述
即插即用的可学习特征生成器G

利用无标记数据中丰富的语义无关信息生成 exemplars 的多样化对应。利用多样化的生成样本来训练分类模型，以保持模型对所有可见类的分类能力。
## 3.2 Learning Framework
形成了两步训练框架：第一步训练 feature generator；第二步借助  feature generator 训练网络
- 为每个类开发一个轻量级的 feature generator
- 当第i个任务结束时，在删除Di之前，我们使用原始数据集Di、示例Mi和未标记数据集Du来学习特定类别的特征生成器。
- ==semantic contrastive learning loss==
- ==Semantic-Decoupling contrastive learning loss==
- ==cycle constraint== 
> 由于从更深层提取的特征图更可能包含任务特定的信息[ 34 ]，因此更深层的特征图更容易遗忘先前的知识。
## 3.3 Anti-forgetting Learning
大的损失包括：
有监督分类损失（有标签数据）、分类损失（生成数据）、知识蒸馏损失
# 4. 前人方法
缓解遗忘问题
- data-driven method
	- 主要关注数据本身、新旧数据之间的关系来缓解遗忘问题
	- distillation 方法
		- locarl[22]、EE2L[4]
	- PODNet [ 6 ]在旧网络的基础上，进一步在网络的不同尺度上对特征表示进行约束。
	- UCIR [ 11 ]使用归一化特征来提取新旧模型之间的特征，而不是原始特征。
	- 使用不同源数据辅助训练：
		- ==GD [ 16 ]==对野外无标签数据进行采样，并定义一个全局蒸馏损失用于防遗忘学习。
		- ==DMC [ 37 ]==使用一个额外的未标记数据集，通过一个双重蒸馏损失来对齐旧的表示和新的表示。
	- 很多 rehearsal-based works 都聚焦于 memory management
		- Mnemonics Training [ 18 ]提出了一种基于元学习的双层优化方法，使得样本具有可训练性。
		- RM [ 3 ]提供了一种新颖的内存管理策略，通过检查在样本中添加噪声后的分类不确定性来选择硬样本。
- structure-driven method
	- 流行的结构驱动方法通过修改网络结构或扩展网络来训练新的任务。
	- BiC [ 32 ]使用一个线性层，通过一个小的样本子集来纠正分类偏差。
	- DER [ 33 ]在每个训练步骤中扩展特征提取主干，并尝试通过可学习的通道级掩码剪枝来减少参数。
	- ITAML [ 21 ]从多个特定任务的模型入手，利用元学习来更好地集成不同的模型。
	- CCGN [ 2 ]为每个卷积层和全连接层插入一个门控结构，以捕获特定类别的知识。
	- RPS-Net [ 20 ]在每一层都定义了并行模块，形成了一个可能的搜索空间，包含了以前的任务特定知识。
	- CEC [ 35 ]设计了一个图模型用于组合来自不同任务的分类器。