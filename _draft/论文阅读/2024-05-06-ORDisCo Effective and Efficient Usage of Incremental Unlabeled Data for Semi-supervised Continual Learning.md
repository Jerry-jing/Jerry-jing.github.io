---
title: ORDisCo Effective and Efficient Usage of Incremental Unlabeled Data for Semi-supervised Continual Learning
author: Jing
date: 2024-05-06
tags:
  - 持续学习
---
> 使用 unlabeled 数据进行半监督连续学习

# 1. 领域问题
领域：类增量问题
问题：
# 2. 创新点
半监督连续学习：semi-supervised continual learning (SSCL)

# 3. 具体方法
## 3.1 问题描述
与有监督的 CL 相比，SSCL 仅提供少量的标记数据和大量的未标记数据
*unlabel 的数据是之前类别的吗？*
## 3.2 方法
Formally, the entire model includes a classifier C, a generator G and a discriminator D.
C：improve its ability of classification and generate pseudo labels for the conditional generation in D and G
D and G： 不断地学习和恢复增量半监督数据中的数据分布。
