---
title: HDRUNet： Single Image HDR Reconstruction with Denoising and Dequantization
author: Jing
date: 2024-04-19
tags:
  - 比赛
---
> 从单张图片中进行 HDR 重建
> 提出了一种新颖的 spatially dynamic encoder-decoder network, HDRUNet，学习具有去噪和去量化功能的单图像 HDR 重建。

[chxy95/HDRUNet: CVPR2021 Workshop - HDRUNet: Single Image HDR Reconstruction with Denoising and Dequantization. (github.com)](https://github.com/chxy95/HDRUNet)
[paper]([arxiv.org/pdf/2105.13084.pdf](https://arxiv.org/pdf/2105.13084.pdf))
# 1. 研究领域和主要问题
研究领域：HDR 重建
主要问题：从单张图片中 HDR 重建
*__HDR 重建的意义__*：
- 传感器的限制：大多数消费级数码相机只能捕捉现实场景中有限范围的亮度
- 成像过程中经常引入噪声和量化误差
# 2. 团队
![[HDRUNet.png]]
# 3. 前人做出的贡献、related work
## 3.1 前人贡献
使用**多张不同曝光图片**解决 HDR 重建
- 缺点：
	- 获取同一场景多张图像难
	- 大多数 HDR 重建的方法忽略噪声和量化损失


## 3.2 related work

# 4. 本文动机、创新点和主要贡献
## 4.1 动机

## 4.2 创新点
充分利用分层多尺度信息的 UNet 式基础网络，包含一个 perform pattern-specific modulation 的 condition network，一个 selectively retraining information 的 weighting network，提出了 Tanh_L1 loss 平衡过曝

## 4.3 主要贡献

# 5. 具体细节、实验

## 5.1 SAMRS Dataset

## 5.2 Backbone Network

## 5.3 Implementation of Multi-Task Pretraining

## 5.4 实践中

## 5.5 Experiments


## 5.6 代码可以借鉴的地方

# 6. 达到的效果
在 [HDR 挑战赛（](https://data.vision.ee.ethz.ch/cvl/ntire21/)[Track1：单帧](https://competitions.codalab.org/competitions/28161)）中获得了第二名NTIRE2021。论文被CVPR2021研讨会接受。
![[HDRUNet.png]]