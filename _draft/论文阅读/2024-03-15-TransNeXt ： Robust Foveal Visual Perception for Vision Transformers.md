---
title: TransNeXt ： Robust Foveal Visual Perception for Vision Transformers
author: Jing
date: 2024-03-15
tags:
  - CVPR2024
  - Backbone
  - 泛读
---
a new visual backbone—— `TransNeXt`：
- 提出了 `Aggregated Attention`，不依赖于堆叠进行信息交换；相似性矩阵不仅仅依赖于 `key ` 和 `query` 的相似性；（`token mixer`）
- 提出了 `Convolutional GLU` ，一种 `channel mixer` 可以弥合 `GLU` 和 `SE` 机制之间的差距，增强了建模能力和模型鲁棒性。（`channel mixer`）
# 1. **领域**
`BackBone`
# 2. **应用**
视觉下游任务： `Domain Generalization on ImageNet-A` 、`COCO object detection、ADE 20K semantic segmentation
> `Domain Generalization`：模型在不同数据分布或不同领域的泛化能力，`ImageNet-A` 包含了大量与原始 `ImageNet` 图像相似但不完全相同的图像。
# 3. **motivation**
由于深度退化现象，许多依赖于堆叠层的 `Vision Transformer` 在信息聚合（`information mixing`）方面做的不好，造成了不自然的视觉感知。
# 4. **效果**
在多种 `model` 大小上达到了 `state-of-the-art` 性能（`ImageNet`、`COCO object detection`、`ADE 20K semantic segmentation`）
# 5. **前人做到的**
解决 `Transformer` 的平方复杂度的问题：提出很多 `sparse attention mechanisms`
- 典型有 `local attention`（将注意力限制在一个 `window` 中），缺点是需要在不同的 `token mixer` 中获得 `cross-window` 的信息交换
- 另外典型的方法是对注意力的 keys 和 values 进行空间上的 `downsample` ，像 `pooling` 和 `grid sampling`。缺点是牺牲了查询对特征图的细粒度感知
- 交替堆叠空间下采样注意力和局部注意力
# 6. 前人缺陷对比
- `local attention`：缺点是需要在不同的 `token mixer` 中获得 `cross-window` 的信息交换
- 对注意力的 `keys` 和 `values` 进行空间上的 `downsample`：缺点是<span style="background:#fff88f">牺牲了查询对特征图的细粒度感知</span>
- ==带有残差快的深度网络表现上像多个浅层网络的 ensemble，表明通过堆叠快实现跨层信息交换不那么有效==
- 因为深度退化现象，许多 `ViT` 模型不能够很好通过堆叠实现 `information mixing`
- `Local attention` 和 `spatial downsampling attention` 与生物视觉有很大不同。生物视觉在视觉焦点处有很高的敏锐度；对各个像素；但是 `local attention` 在窗口边缘和中心的像素点就会被区别对待。
# 7. 具体细节
- `Pixel-focused Attention`：双路径设计，一个是有细粒度的 `attention`（`nearest neighbor features`），一个是有粗粒度的 `attention` （`global perception`）。可以有效模拟眼球运动。    
- 将 `query embedding` 和 `positional attention mechanisms` 合并到 `pixel-focused attention` 中，导致 `Aggregated Pixel-focused Attention`。使得相似度矩阵不仅仅依赖于 `query` 和 `key` 的相似度。
- 重新设计 `channel mixer` ，提出了新的叫 `Convolutional GLU` 的 `channel mixer`，更适合图像任务，集合了基于局部特征的注意力通道来增强模型鲁棒性。
# 8. 学到的一些东西
- `Foveal` 在这里指的是视觉系统中的中央凹区域，是人眼视网膜上具有最高视觉分辨率和最敏锐视力的部分。在视觉研究中，鼻窝视觉感知指的是模仿人类视觉系统中这种高分辨率的中央凹区域的感知机制。
- `ViT` 模型由两个主要部分组成：自注意力层（`token mixer`）和 `MLP` 层（` channel mixer `）
- `Self-attention` 机制通过计算 `queries` 和 `keys` 之间的相似度生成相似度矩阵，在特征提取上扮演重要角色，在特征提取上有很大潜力。
- `Transformer` 中的 `token` 通常是指输入模型的最小单位
- `Transformer` 由于是对 ` NLP ` 设计的，在视觉任务上有固有的问题：平方复杂度、高内存消耗。
- 不是 `state-of-the-art`，为什么也能中
# 9. 计算机视觉特有词汇总结
- `Affinity matrix`：亲和矩阵，就是相似度矩阵
- `Inductive bias`：归纳偏置，学习过程中对数据的先验假设和偏好
- `Receptive field`：感受野