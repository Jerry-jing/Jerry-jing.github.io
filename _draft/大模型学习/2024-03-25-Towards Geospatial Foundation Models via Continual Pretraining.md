---
title: Towards Geospatial Foundation Models via Continual Pretraining
author: Jing
date: 2024-03-25
tags:
  - foundation_models
  - MAE
---

# 1. 领域 
在遥感领域的 `foundation model` 
# 2. 前人做到的
在 `geospatial applications` 中引入 `foundation models` 的两种主要方式：
- 利用现有的自然图像领域的基础模型直接微调
	- 优点：有效
	- 缺点：自然图像和遥感图像的 `domain gap`
- 预训练地理空间领域的模型
# 3. 论文创新点
## 3.1 `GeoPile` 数据集
简洁、多源的数据集
## 3.2 多目标持续预训练范式
> ==teacher-student strategy== with both a ==distillation objective== and ==self-supervised masked image modeling==

# 4. 实验
