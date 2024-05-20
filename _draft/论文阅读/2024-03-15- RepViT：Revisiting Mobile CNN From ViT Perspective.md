---
title: RepViT：Revisiting Mobile CNN From ViT Perspective
author: Jing
date: 2024-03-15
tags:
  - CVPR2024
  - 轻量化
  - Backbone
  - 泛读
---
# 1. **领域**
轻量级模型（受限于移动设备资源有限），延时准确率之间的 `tradeoff`
> 轻量级模型中的“轻量”通常指的是模型在参数数量、计算复杂度和内存消耗等方面相对较低的特性。
# 2. **应用**
边缘部署
# 3. **motivation**
从 `ViT` 的角度重新看轻量级的 `CNNs`，`MobileNetV3` 整合轻量级 `ViTs`，得到 `RepViT`
# 4. **效果**
超过了现在 `state-of-the-art` 的轻量级 `ViTs`，在各种任务中（`ImageNet` 图像分类，COCO-2017实例分割目标检测，`ADE20k` 语义分割）表现出少的延时；`ReViT-SAM` 推理速度比现在先进的 `MobileSAM` 快 10 倍。
# 5. **前人做到的**
- `CNN` 方面：有效结构：`separable convolutions` , `inverted residual bottlenecks`, `channel shuffling` , and `structural re-parameterization`
- `ViT` 方面：`CNN` 与 `ViT` 混合；`novel self-attention operations with linear complexity and dimension-consistent design principles`。
# 6. **`ViT` 做轻量级方面的缺陷**
比 `CNN` 更受高分辨率输入的影响（应该是与计算方式有关，与输入的时间复杂度）
# 7. **轻量级 `ViT` 和轻量级 `CNN` 的对比**
- 结构相似性：都使用卷积模块学习空间局部表示；但是轻量级 `CNNs` 通常采用扩大卷积核来学习，轻量级 `ViTs` 通常采用多头注意力模块
- 轻量级 `ViTs` 采用 `MetaFormer` 结构，轻量级 `CNNs` 采用 `inverted residual bottlenecks` 结构
# 8. **`RepViT`**
一组轻量级 `CNNs`，完全由类似于 `MetaFormer` 的结构组成
# 9. 学到的一些东西 
过去的东西结合现在的东西，`CNN` 结合 `ViT`，`CNN` 借鉴 `ViT`