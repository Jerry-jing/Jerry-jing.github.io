---
title: Yann Lecun 专访
author: Jing
date: 2024-03-19
tags:
  - 名人访谈
---
# 1. `LLM is useful, but not interesting`
- 我们学到的大部分内容和只是都是通过*我们的观察与现实世界的互动*，而不是通过语言。
- `LLM` 缺乏对物质世界的理解 （有更大的抽象存在于语言之前并映射到语言）
	- 与它的训练方式有关，掩掉一个词，去生成这个词，得到的只是一个概率分布，而不是 `learning` 和 `reasoning` 的能力。（why called `Autoregressive LLM`）

# 2. `Lecun` 认为
- 通过*预测*来构建这个世界是肯定的，但是通过语言来构建是否定的。
- 语言包含的信息还是不够的。
- `Lecun` 他们尝试了很多次用给模型（`straight neural nets`、`GANs`、`VAE` 、`kind of regularized autoencoders` 等）送入视频，让它预测之后会发生的事情，都失败了；但是 `LLM` 却成功了。
	- `self-supervised by reconstruction` ： 通过损坏的图像中重建出良好的图像行不通，它提到了 ` MAE ` 的工作。训练系统重建图像并不会让它学习到图像的良好通用特征。
	- `joint embedding`：

`mental models`
`Facebook AI Research (FAIR)`