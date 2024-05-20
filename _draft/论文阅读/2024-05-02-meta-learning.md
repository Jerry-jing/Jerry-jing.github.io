---
title: meta-learning
author: Jing
date: 2024-05-02
tags:
---
 meta-learning: learn to learn
 machine-learning: looking for a function

# 1. meta-learning
- 输入 Training Examples ，输出 classifier（一个 function）。
- 学的东西可能包括：net architecture, initial parameters, learning rate 等
- 训练资料：training task
	- 假设想要训练一个二元的分类器，需要准备很多二元分类的任务（假如 task1：apple & orange；假如 task2：car & bike；每个 task 中都有 train 和 test）
- loss function：用测试资料上做 loss
- optimization approach：
	- 用梯度计算
	- 不能用梯度计算时（如学习网络架构时，或者值为离散的），使用 reinforcement learning / evolutionary algorithm
> few-shot learning 和 meta-learning 还是有一定区别的，meta-learning 期望学会学习，few-shot learning 期待用更少的数据去学习
![[meta-learning.png]]

# 2. meta-learning 和 machine learning 的区别

machine learning：find a function f
- 如猫狗分类器，输入是图片，输出是类别
- 有 hand-crafted alogrithm( f )
- within-task training
meta learning: find a fuction F that finds a fuction f
- 输入是很多 task 的 dataset，输出是一个 f
- acorss-task training 包含多个任务的 within-task training 和 within-task testing
- acorss-task testing 包含 within-task training 和 within-task testing
- within-task training 和 within-task testing 两个流程合起来叫做一个 episode 
> MAML 中把 acorss-task training 叫做 outer loop in "learning to initialize"；把 within-task training 叫做 inner loop in "learning to initialize"

# 3. meta-learning 和 machine learning 的相同点
- overfitting on training tasks 的问题
	- 解决方案：get more trianing tasks to improve performance、task augmentation
- There are also hyperparameters when learning a learning algorithm……
- development task
