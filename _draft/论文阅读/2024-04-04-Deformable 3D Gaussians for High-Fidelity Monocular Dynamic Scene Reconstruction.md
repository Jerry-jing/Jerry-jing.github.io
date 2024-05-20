---
title: Deformable 3D Gaussians for High-Fidelity Monocular Dynamic Scene Reconstruction
author: Jing
date: 2024-04-04
tags:
  - CVPR2024
  - 3D
---
# 1. 研究领域和主要问题
领域：3D 重建（单目动态场景建模）
领域应用场景：增强现实、虚拟现实、3D 内容制作和娱乐
主要问题：解决之前 Implicit neural representation 所遇到的一些问题
# 2. 团队
单位：浙江大学CAD & CG 国家重点实验室、字节跳动公司
导师：CVPR 后面没有导师信息
# 3. 前人做出的贡献
Implicit neural representation
缺点：尖端的动态神经渲染方法在很大程度上依赖于这些隐式表示，它们经常难以捕捉场景中对象的复杂细节；很难在一般动态场景中实现实时渲染，限制了它们在各种任务中的使用。

# 4. 本文创新点
deformable 3D Gaussians Splatting method：可变形 3D 高斯分布方法
- 解决 Implicit neural representation 有的问题
annealing smoothing training mechanism with no extra overhead：没有额外开销的退火平滑训练机制
- 减轻不准确姿势对显示数据集中时间插值任务平滑度的影响


# 5. 实验
硬件设备
数据集是否很大
花费时间是否很多

# 6. 达到的效果
在渲染质量和时间性能方面优于 `HyperNeRF`
可以获得更高的渲染质量，实现实时的渲染速度
非常适合以下的任务：novel-view synthesis、time interpolation、real-time rendering

纯理论方向，比如优化方法，激活函数，可解释性等，一般只需要在mnist,cafir等小数据集上证明可行性就OK。但这也是难度最大的方向

计算机视觉不止有深度学习
3d slam 几何
基于光学，成像相关的图像处理
都比调参硬核，有的给你张纸就可以开始算了