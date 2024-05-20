---
title: Alpha-CLIP： A CLIP Model Focusing on Wherever You Want
author: Jing
date: 2024-03-15
tags:
  - CVPR2024
  - CLIP
  - 泛读
---
# 1. **领域**
`CLIP`、预训练
# 2. **应用**
在多种任务上效果很好，如 `open-world recognition`（识别数据集中未出现的数据）、`multimodal large language model` 和 `conditional 2 D ` / `3 D generation`
# 3. **motivation**
`CLIP` 通过将文本和图像两种模态对齐去理解整个图片（包括所有细节），但是做下游特定任务对特定区域理解至关重要，可以通过人为给定 `points`、`masks`、`boxes`，或者通过 `perception models`（模拟人类感知过程的模型）。（就像 `SAM`、`GLIP`、`proposal networks`）
# 4. **效果**
- 在图像识别方面：保持了原始 `CLIP` 视觉识别的能力，而且能够激发基于区域识别的能力。这种 region-based recognition 的能力可以帮助像 `Referring Expression Comprehension（REC）` 的任务，或者作为 `Open Vocabulary Detection（OVD）` 的 `data engine` 使用。
- 多模态学习框架中的视觉 `backbone`：可以促进区域级字幕、`VQA`。可以减轻幻觉的发生，减少模型偏见。
- `2D` 生成：可以增强 `BLIP-Diffusion` 的控制能力。
- `3D` 生成：能够有效的与 `diffusion model` 结合，提高 `3D` 对象生成的质量；还能够与 `NeRF` 一起使用，创建更优越的 `3D` 对象。
# 5. **前人做到的**
通过两种策略获得 `region-focused CLIP features`
- 裁剪出感兴趣的区域；给不感兴趣的图片部分、`features`、`attention mask` 打 `mask`
- 使用 `circles` 和 `mask contour` 突出感兴趣区域。
# 6. 前人缺陷、对比
- 裁剪感兴趣的区域：会 `disrupt（in cropping）` 和 `omit （in masking ）`
- 使用 `circles` 和 `mask contour `突出感兴趣区域：对用户友好，但是会导致识别和生成结果不好（加入了之前没有的东西）
# 7. 具体细节
有一个辅助的 alpha 通道去建议关注特定的区域；而且在百万级别的 `RGBA region-text pair` （数据集是用 SAM 和 BLIP-2 生成的）中微调。
# 8. Limitation 
- 用户无法自己指定 attention 的强度；
- Alpha-CLIP 和原始 CLIP 一样都是低分辨率，阻碍 Alpha-CLIP 识别小物体。
# 9. 学到的一些东西 
效果好就可以发，很 6
# 10. 计算机视觉相关词汇
- `plug-and-play methodology` ：即插即用的方式
- `Referring Expression Comprehension(REC)` ：是指理解和解释指向性表达
- `Open Vocabulary Detection (OVD)`：检测到原本不在预定义类别列表中的物体
- `MLLM（Multimodal Learning Framework）`：多模态学习框架