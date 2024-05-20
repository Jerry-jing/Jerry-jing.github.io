---
title: CL论文合集
author: Jing
date: 2024-05-07
tags:
  - 持续学习
---
# 1. blurry task
## 1.1 Online Continual Learning on Hierarchical Label Expansion
==multi-level hierarchical class incremental task== 
> hierarchical classification 是一种多标签分类任务，其中分类任务被组织成一个层次结构。一个对象可以属于多个类别，这些类别存在上下级关系。
- ==提出新的 CL setup：HLE==，**new online class-incremental hierarchyaware, task-free，with an online learning constraint**
	- 包括两个 scenarios：single-depth、multiple-depth scenarios![[HLE_setup.png]]
	- single-depth scenario，在第一个任务中对所有父类进行学习，通过后续任务对其进行扩展
	- dual-label：overlapping data
	- single-label：disjoint data
- ==提出新的 online CL method：PL-FMS==，consist of pseudo-labeling (PL) based memory management and flexible memory sampling (FMS) 
	- 分类：rehearsal-based incremental learning approach
	- 组成：Pseudo-Labeling based Memory Management、Flexible Memory Sampling![[PL-FMS.png]]
	- PL：与层次有关得去去除内存中的数据
	- FMS：balance the usage of memory and data stream 可能是随着时间的增加，拿到以前类的概率会增加
- ==state-of-art==：在 CIFAR 100, StanfordCars, iNaturalist-19, ImageNet-Hier100 上取得不错的效果
	- CIFAR100 和 Stanford Cars 数据集各自有 2 个层次
	- 评估 HLE setup 中 multiple-depth scenario：CIFAR100 和 iNaturalist19 
	- 与 rehearsal-based 方法比较
		- 与 conventional CL setup 下的 ER、EWC++、MIR 方法对比；（MIR 方法没有做 multiple-depth scenario）
		- 与 recently proposed CL setup 下的 RM 和 CLIB 对比；
	- 与 regularization-based 方法比较
		- 与 BiC 和 GDumb 方法对比（GDumb 方法没有做 multiple-depth scenario）
思路：构建类之间的标签关系，允许标签空间有更多结构（即不同的层次）
hierarchy level 作为模型的软提示（soft hint）给出
## 1.2 Online Continual Learning on Class Incremental Blurry Task Configuration with Anytime Inference
- 提出新的 CL setup：i-Blurry，**online, task-free, class-incremental, blurry task boundaries, and subject to inference queries at any moment**
	- i-Blurry-N-M split：有 disjoint, 也有 blurry
- ==提出新的 online and task-free CL method==：**new memory management，memory usage，and learning rate scheduling startegy**
	- baseline：for the memory management policy, reservoir sampling; for memory usage, experience replay; for LR scheduling, exponential LR schedule but reset the LR when a new class is encountered
	- ==sample-wise importance based memory management==：丢弃损失减少最少的样本，即对训练最无用的样本
		- 两类型的 sample 具有更重要的得分：一个 batch 训练后 loss 减少了；一个 batch 训练后 loss 增加了
	- ==memory only training==
		- 只使用来自内存的样本进行训练，而不使用流式样本
		- 作者认为直接使用流式样本会使训练偏向于最近的样本，memory 对流式样本起到分布稳定器的作用
	- ==adaptive learning rate scheduling==
		- exponential with reset scheduler：遇到新类会让 LR 重置为初始值
		- data-driven LR scheduling scheme：根据 LR 在内存上的优化程度，以数据驱动的方式自适应地改变 LR
- outperforme existing CL models
- 提出新的指标来衡量 CL 模型实现所需的随时推理能力

**parameter isolation methods**
regularization methods
replay methods
**RM**
## 1.3 Online Class Incremental Learning on Stochastic Blurry Task boundary via Mask and Visual Prompt Tuning
> i-Blurry 的局限性：每个任务出现相同数量的新类；新类和模糊类在每个任务中所占的比例相同
- ==提出 incremental learning scenario，Si-Blurry==，当任务边界随机变化时，神经网络不断地在线学习新的类别
- 提出防止模型的任务内和任务间的遗忘：instance-wise logit masking、contrastive visual prompt tuning loss
	- instance-wise logit masking：learnable mask paired with prompts ( learn more intra-relevant and easier learning goals )
		- query：feature extracted by the pretrained model
		- each prompt can be responsible for a certain region of the feature space in which classes have similar extracted features.
		- 使用元素相乘将 mask 应用于 logit，然后计算交叉熵损失
	- contrastive visual prompt tuning loss
- 提出解决类别不平衡问题：gradient similarity-based focal loss and adaptive feature scaling
- overwhelming performance：在 Si-Blurry 的场景下，在 CIFAR-100、Tiny-ImageNet、ImageNet-R 取得很好的性能

# 2. 基于改进模型的


# 3. 自己的想法 
可以看作是“自监督对比学习”的一种改进
有监督对比学习（Supervised Contrastive Learning）：提取好的 feature
- 自监督对比学习（自动产生一种标签）：“是否来源于同一张图片”
- 监督对比学习（Supervised Contrastive Learning）：“是否属于同一个类”

cross-entropy loss
缺点：对噪声标签缺乏敏感、边缘较差，从而导致泛化性能下降。
但是其他替代方案在大规模数据集上效果并不优越

- 提出 SupCon 损失（对比损失函数的扩展，允许每个 anchor 有更多的正值）
	- 比 cross-entropy with standard data augmentations 效果好
	- 方法：首先两次数据增强获得该批次的两个副本，两个副本都通过编码器网络前向传播获得 2048 维归一化嵌入，训练期间经过投影网络进一步传播，投影网络在推理时被丢弃。监督对比损失根据投影网络的输出进行计算。为了使用经过训练的模型进行分类，我们使用交叉熵损失在冻结表示之上训练线性分类器。
	- Data Augmentation module
	- Encoder Network：map **x** to a representation vector **r**
	- Projection Network：map **r** to a vector **z**
- 为许多数据集的 top-1 accuracy 提供了一致的提升
- analysis：损失函数的梯度从 hard positives 到 hard negatives 中学习；实践表明，得到的损失对一系列超参数的敏感性不如交叉熵

