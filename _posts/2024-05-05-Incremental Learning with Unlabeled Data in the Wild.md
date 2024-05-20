---
title: Incremental Learning with Unlabeled Data in the Wild
author: Jing
categories: [论文阅读]
date: 2024-05-05
tags: [持续学习]
img_path: /assets/img/
---

> 利用野外连续且大量的==无标签数据==来缓解类增量学习中的灾难性遗忘问题。
# 1. 领域问题
领域：类增量学习
问题：利用无标签数据解决类增量学习的灾难性遗忘问题
问题来源：我们生活在一个连续和大量的数据流中，大量的未标记数据很容易在即时或短暂的情况下获取
问题 setup：
- unlabel 的大部分与感兴趣的任务无关
- 不是 blurry task 的任务
# 2. 创新点
- 提出新的训练损失，全局蒸馏( global distillation )，利用数据来有效提取先前任务的知识
- 3 步学习方案提高全局蒸馏的有效性
	- 训练专门从事当前任务的教师；
	- 通过第一步蒸馏中学习到的前一个模型和教师的知识来训练模型；
	- 微调以避免对当前任务的过拟合；
- 提出了一种带有置信度校准模型的采样方案( sampling scheme with a confidencecalibrated model )，以有效地利用大量的未标记数据流
# 3. 具体方法
## 3.1 类增量学习描述
构建数据集的方式：基于置信度的采样策略
全局蒸馏方式

增量式多任务学习（Incremental Multi-Task Learning, IMTL）
有标签数据：有监督的 loss
未标记数据：蒸馏损失
置信度损失来使模型的置信度校准
## 3.2 三步学习的全局蒸馏
置信度损失 + 分类损失联合最小化 -> 使模型的置信度校正达到采样目的
没看懂公式
## 3.3 对外部数据集( External Dataset )进行抽样
- 前提：data in wild 的大部分与感兴趣的任务无关，需要采样
- 目的：
	- 缓解灾难性遗忘，对先前任务中预期的外部数据进行采样，使训练数据集达到平衡
	- 使模型的置信度得到校准，需要对一定量的 out-of-distribution 数据进行采样
- 方法：
	- 在每个阶段开始时，从未标记的数据流中随机采样未标记数据作为 OOD，并根据 P 的预测为先前任务中的每个类采样最可能的数据
# 4. 实验
与 LwF、DR、E2EiL 对比，ours 方法叫 GD ( global distillation )

# 5. 处理 unlabel 的数据
- 从 unlable 的数据流中随机采样 OOD 的数据
- 根据 P 的预测为先前任务中的每个类采样最可能的数据（*有置信度？*）


GD 根据存储的实例将旧网络中的知识提取到新网络中。