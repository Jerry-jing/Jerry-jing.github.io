---
title: iCaLR Incremental Classifier and Representation Learning
author: Jing
date: 2024-05-10
tags:
  - 持续学习
---

iCaRL 的三个组件：最近均值样本分类器（a nearest-mean-of-exemplars），对数据表示的变化具有鲁棒性，每个类仅需要存储少量样本；基于羊群的步骤（a herding-based step），用于优先选择样本；表示学习步骤（a representation learning step）将 exemplars 与 distillation 结合


# 1. 领域问题

# 2. 团队
单位：University of Oxford/IST Austria、Christoph H. Lampert IST Austria
作者：Sylvestre-Alvise Rebuffi、Alexander Kolesnikov、Georg Sperl
# 3. 

# discussion
use of exemplar images
无法存储原始数据，一个可能的方向是通过自动编码器隐式编码早期任务的特征。