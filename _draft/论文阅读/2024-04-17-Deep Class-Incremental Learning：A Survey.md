---
title: Deep Class-Incremental Learning：A Survey
author: Jing
date: 2024-04-27
tags:
  - 比赛
  - 持续学习
  - 方向探索
  - 类增量学习
---
> Class-Incremental Learning(CIL) 三种分类：data-centric, model-centric, algorithm-centric.
> stability-plasticity dilemma

address the incremental learning problem:
- class-incremental learning
- Task-Incremental Learing：会给每个任务的标签，模型不需要有区分不同任务的能力
- Domain-Incremental Learning：标签处于同一空间
在深度学习火之前也有类增量学习这一概念。
[19] 2020 年的类增量的综述

Table 1：以数据为中心、以模型为中心、以算法为中心的类增量学习
Figure 3：类增量学习路线图

# 1. 定义
## 1.1 问题表述
- 类增量学习的定义
- Class Overlapping：
	- blurry class-incremental learning( Blurry CIL )：模糊类增量学习，在不同的 task 种类别有重叠
	- ==本文关注没有重叠类的典型 CIL 设置==
	- CIL 模型分为 embedding module 和 linear layers。 （linear layers 映射到标签个向量上，之后进行 softmax 变为概率）
- Online CIL：
# 2. 类增量学习分类
## 2.1 Data-Centric Class-Incremental Learning
> Data-Centric methods 专注于用 exemplars 解决 CIL，进一步分为 data replay 和 data regularization。
- ==Data Replay==：网络可以通过重新访问以前的样本来克服灾难性遗忘问题
	- 保存一个额外的样本集，并将其包含到模型更新过程中
		- 实例采样过程：[39]建议对具有高预测熵（high prediction entropy）且靠近决策边界的样本进行采样；
		- replay these ‘hard’ exemplars 时，[35]通过数据增强来估计样本的不确定性;
		- [40]提出在线增量学习中使用贪婪策略对样本进行采样，证明了样本选择相当于以参数梯度为特征最大化样本多样性；
	- 解决重放时的消耗巨大的 memory cost 的问题，用来构建内存高效的 replay buffer
		- [46]建议将特征保存在样本集中；
		- [47]建议保留低保真图像而不是原始图像以节省内存；、
	- 分类：
		- direct replay
		- generative replay：存在于两种模型（用于数据生成的生成模型、用于预测的分类模型）
- ==Data regularizaiton==：用以前的数据对模型进行正则化并控制优化方向
	- GEM [59] 在优化新模型的同时，针对保存的额外样本集，需要使样例集计算的损失不能超过之前的模型
	- Adam-NSCL [61]
	- OWM [62]
	- LOGD [63]
- Data-Centric Methods 广泛应用于基于图像的相机定位、语义分割、视频分类和动作识别
- 由于样本集只保存了训练集的一小部分，数据回放可能会遇到过拟合问题；由于少样本和多样本之间的差距，也会出现数据不平衡问题。可以通过平衡采样解决这个问题。
- 数据正则化依赖于特定的假设，例如将样本的丢失视为遗忘的指标，另一方面，数据正则化方法依赖于特定的假设，例如将样本的丢失视为遗忘的指标。这些假设在某些情况下可能不成立，从而导致性能不佳。
## 2.2 Model-Centric Class-Incremental Learning
> Model-Centric methods 要么规范模型参数以免漂移，要么扩展网络结构以获得更强的表示能力。
- ==Dynamic Networks==：动态的调整模型的表示能力以适应不断变化的数据流
	- neuron expansion：DEN[64]、RCL[65]、NAS[142]
	- backbone expansion：PNN[67] 为每个新任务学习一个新 backbone，新旧模型之间有分层连接，可以重用以前的功能、DER[20]、P&C[69]、FOSTER[21]、MEMO[22]
	- prompt expansion：DyTox 每个新任务只扩展任务 token、L2P[72]、DualPrompt[73]、CODA-Prompt[74]、S-Prompt[75]
- Parameter Regularization：认为每个参数对任务的贡献是不相等的
	- EWC[77] 有监督训练新类的同时，让模型参数尽可能靠近（重要性矩阵 omega）、RWalk[39]、IMM[8]、IADM[82]、CE-IDM[83]、K-FAC[84]
- 针对网络掩码工作，将大型网络划分为每个任务的子网络，但是决定激活各个子网络需要任务 ID 或者学习额外的任务分类器
## 2.3 Algorithm-Centric Class-Incremental Learning
> Algorithm-Centric methods 要么利用知识蒸馏来防止遗忘，要么纠正 CIL 模型中的偏差。
- ==Knowledge Distillation==
	- logit distillation：LwF[85]  [86] 利用知识蒸馏构建正则化项防止遗忘、iCaRL[32] 、D+R[88]、GD[89]、DMC[90]、ABD[91]、COIL[92]、
	- feature distillation：UCIR[93]、AFC[97]、PODNet[98]、CVIC[162]、DDE[99]、GeoDL[100]
	- relational distillation：（对齐新旧模型之间的输入关系）R-DFCIL[101]、ERL[102]、TPCIL[103]、TOPIC[104]、MBP[105]
- Model Rectify：旨在减少 CIL 模型的归纳偏差
	- UCIR[93]、WA[115]、SS-IL [116]、RPC [117]、E2E [112]、BiC [87]、SDC [107]、CwD [108]、ConFiT [109]、CCLL [110]、RKR [111]、FACT [106]
- 知识蒸馏效果没有动态网络效果好，但是占用内存小。
