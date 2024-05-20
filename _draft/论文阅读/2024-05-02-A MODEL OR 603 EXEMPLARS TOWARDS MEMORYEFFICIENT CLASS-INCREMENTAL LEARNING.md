---
title: A MODEL OR 603 EXEMPLARS TOWARDS MEMORYEFFICIENT CLASS-INCREMENTAL LEARNING
author: Jing
date: 2024-05-02
---
> 摘要：
> 类增量学习需要在内存有限的情况下没有灾难性遗忘的问题。防止遗忘的方法有：保存之前样本；保存之前历史模型，他们发现保存之前历史模型会使分类效果提升，但是他们没有考虑到存储模型的内存预算，这其实是不公平的。将模型大小计入预算发现保存历史模型并不总是有效的。
> 我们做的：
> 我们在不同的内存规模下全面评估不同的 CIL 方法，同时兼顾准确性和内存大小；
> 我们==研究内存缓冲区==的构造提高内存效率，分析网络中不同层的效果，发小 CIL 中浅层和深层具有不同特征。（共享浅层并仅为新任务创建深层）
> 我们提出一个基线——MEMO Memory-efficient Expandable MOdel，==MEMO 基于共享的广义表示扩展了专用层==，适度的成本有效提取不同的表示并维护了代表性样本
# 1. 领域问题
领域：类增量学习
问题：在内存有限的情况下（包括存储样本和存储模型），如何提高分类性能？
# 2. 知识扫盲
- CIL 算法的性能上限，所有流数据进行离线训练，但是需要无限的内存预算。
- CIL 的内存成本来源：exemplar buffer，model buffer
- How to fairly compare different methods? 将 extra backbones 和 extra examples 进行转化，如：ResNet32 模型与 CIFAR100 的 603 个图像具有相同的内存大小，而 297 个 ImageNet（Deng 等人，2009）图像与 ResNet18 主干网具有相同的内存大小。
- exemplar-based：seek to rehearse former knowledge when learning new
	- replay
	- iCaRL：build konwledge distillation regularization with exemplars to align the predictions of old and new models
	- treats the loss on the exemplar set as an indicator of former tasks' performance and solves the quadratic program problem as regularizaiton：Gradient episodic memory for continual learning（NeurIPS）
	- utilize examplars for balanced finetuning or bias correction：Large scale incremental learning（CVPR）、End-to-end incremental learning（ECCV)
	- 通过额外的生成模型生成新样本
- model-based：save extra model components to assist incremental learning
	- 增量的增加模型

Memory-efficient incremental learning through feature adaptation（ECCV 2020)：保存提取的特征而不是原始图像
Memory efficient class-incremental learning for image classification（TNNLS 2021）：保存低保真样本以降低内存成本
Always be dreaming: A new approach for data-free class-incremental learning（ICCV 2021）、Dual-teacher class-incremental learning with data-free generative replay（ICCV 2021）、Data-free knowledge distillation for deep neural networks（2017）：通过无数据知识蒸馏来释放范例的负担

两种 Baseline 的方法：
the examplar-based method：结合交叉熵损失和知识蒸馏损失
the model-based method：单个 backbone 无法将描述新类的动态特征。
DER  Dynamically expandable representation for class incremental learning（CVPR 2021） ：为每个任务添加一个 backbone 来描述新任务的新特征

a good CIL method with extendability should work for any memory cost.
# 3. 创新点
- 提出了 MEMO 基线
- 提出了几种新的衡量标准——同时考虑性能和内存大小
- 探讨内存缓冲区存什么效果好——共享浅层并仅为新任务创建深层有助于节省 CIL 中的内存预算
	- motivation：存储模型效果好，但我希望存储更高效，省下的内存可以用来存 exemplars
	- block-wise gradient：看梯度，深层在新任务中调整更多
	- block-wise shift
	- feature similarity
# 4. 实验
- Dataset：CIFAR100、ImageNet100/1000
- Dataset split：Base-x，Inc-y（x：base classes；y：new classes）
- model：ResNet32 for CIFAR100 and ResNet18 for ImageNet
- 该模型以 128 的批量大小训练 170 个 epoch，并使用带有动量的 SGD 进行优化。学习率从 0.1 开始，在 80 和 150 epoch 时衰减 0.1。
- 我们使用羊群算法（Welling，2009）从每个类别中选择样本。
HOW TO FAIRLY COMPARE CIL METHODS?
