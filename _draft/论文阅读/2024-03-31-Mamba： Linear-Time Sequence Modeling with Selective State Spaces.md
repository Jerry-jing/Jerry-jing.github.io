---
title: Mamba： Linear-Time Sequence Modeling with Selective State Spaces
author: Jing
date: 2024-03-31
tags:
  - CVPR2024
  - 泛读
  - Backbone
---
现在的基础模型几乎普遍基于 Transformer 架构和它的核心注意力模块。Transformer 在长序列上的计算效率地下（之前的解决方案有 linear attention, gated convolution, recurrent models, structured state space models / SSMs，关键弱点是无法执行基于内容的推理）

A general sequence model backbone ,Mamba
作者的写作思路：Foundation models 和 在下游任务 finetune 是一种很好的范式，FMs 的 backbone 是与模型架构无关的，现在人又喜欢用 transformer，我要创新
# 1. 领域

# 2. 前人做的与对比
## 2.1 transformer
transformer 好处：the efficacy of self-attention is attributed to its ability to route information densely within a context winodow, allowing it to model complex data.
transformer 坏处：However, this property brings fundamental drawbacks: an inability to model anything outside of a finite window, and quadratic scaling with respect to the window length.
## 2.2 SSMs
SSM, a class of architectures for sequence modeling.
可以解释为递归神经网络（RNNs）和卷积神经网络（CNNs）的组合，灵感是经典状态空间模型（classic state space models）
SSM 好处：计算有效率，与序列长度呈线性关系

# 4. 创新点
- 简单地让 SSM 参数作为输入的函数，可以解决其离散模态的弱点，允许模型根据当前标记选择性地沿序列长度维度传播或忘记信息
- 这种变化使高效卷积不能使用，本文设计了一种硬件感知的并行算法。

将这种选择性的 SSM 集成到简化的端到端神经网络架构中，不需要注意力甚至是 MLP 模块。
# 5. 优点
推理快速：是 Transformers 吞吐量的 5x，序列长度线性缩放
state of the art：在多种模态——language、audio、genomics 上达到 state-of-the-art 的效果
# 6. 感想


