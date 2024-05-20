---
title: ExpandNet ：A Deep Convolutional Neural Network for High Dynamic Range Expansion from Low Dynamic Range Content
author: Jing
date: 2024-04-21
tags:
  - 比赛
  - Image_processing
---
> 预告：ExpandNet 接受 LDR 图像作为输入，通过==端到端的方式==生成 HDR 图像，==不使用上采样==。
> 模型尝试重建因为 quantization、clipping、tone mapping、gamma correction 而丢失的信息
> 方法是全自动、数据驱动的，不需要任何启发或者专业知识。
> 比 expansion / inverse tone mapping operators 相比更好
# 1. 方法
在局部范围内，网络的一个分支学习如何维护和扩展高频细节；
扩张分支学习更大像素邻域的信息；
最后第三个分支通过学习输入的全局上下文来提供整体信息。

这个架构旨在避免对下采样特征进行上采样，试图减少更直接的方法（如自动编码器架构）可能产生的 blocking 、haloing artefacts。
## 1.1 网络结构
> 前人有人认为频繁使用反卷积层，会导致棋盘伪影；上采样会导致上下文缺失的区域出现不必要的信息泄露，如大面积过度曝光区域。
![[ExpandNet architecture.png]]
- 三分支架构：每个分支都是 CNN
	- local branch：处理 local detail，避免使用上采样和下采样，小感受野提供像素级学习，保留高频特征。
	- dilation branch：处理 medium level detail，避免使用上采样和下采样
	- global branch：higher level image-wide features，仅使用下采样，降低输入的维度并捕获抽象特征，有足够大的感受野。
- 三个分支的输出被融合并由一个小的最终卷积层进一步处理，生成预测的 HDR 图像
- 注意：输入的 LDR 和预测的 HDR 都在 [0, 1] 范围内
- 三分支的作用：local branch 产生高频特征，dilation branch 添加中频特征，global branch 通过添加低频并调整图像的整体清晰度改变整体外观
![[ExpandNet three branches contribution.png]]
## 1.2 Fusion
- local output 和 dilation output 具有和输入相同的高度和宽度，沿特征图维度连接。
- global output 是包含 64 个特征的向量，该向量沿宽度和高度维度进行复制，用来匹配其他两个输出的维度
- 将 local output、dilation output、global output 沿特征图维度连起来，总共产生 256 个特征。
- 用 1x1 的卷积，它将 global output、local output 和 dilation output 的每个单独像素融合，从而组合来自多个尺度的上下文
- 融合层的输出由具有 3 × 3 内核、步长 1 和填充 1 的最终卷积层进一步处理。
## 1.3 Activations
除输出层外，所有层均使用 SELU 激活函数，是 ELU （Exponetial Linear Unit）的变体。输出层用 sigmoid 函数。
## 1.4 Loss function
> 论文中提到了频繁使用 L2 距离会导致图像模糊。

损失函数 L 是数据集中的预测图像和真实图像之间的 L1 距离。添加额外的余弦相似度项以确保每个像素的 RGB 向量的颜色正确性。
![[ExpandNet loss.png]]
```python
class ExpandNetLoss(nn.Module):
    def __init__(self, loss_lambda=5):
        super(ExpandNetLoss, self).__init__()
        self.similarity = torch.nn.CosineSimilarity(dim=1, eps=1e-20)
        self.l1_loss = nn.L1Loss()
        self.loss_lambda = loss_lambda
  
    def forward(self, x, y):
        cosine_term = (1 - self.similarity(x, y)).mean()
        return self.l1_loss(x, y) + self.loss_lambda * cosine_term
```
对于为什么使用余弦相似度，没有看懂这是为什么
```text
Cosine similarity measures how close two vectors are by comparing the angle between them, not taking magnitude into account. For the context of this work, it ensures that each pixel points in the same direction of the three dimensional RGB space. It provides improved color stability, especially for low luminance values, which are frequent in HDR images, since slight variations in any of the RGB components of these low values do not contribute much to the L1 loss, but they may however cause noticeable color shifts.
```
## 1.5 训练细节
### 1.5.1 数据集
创建了一个 HDR 图像数据集，其中包含 1013 个训练图像和 50 个测试图像，分辨率范围从 800×800 到 4916 × 3273。这些图像是从各种来源收集的，包括内部图像、来自HDR 视频和网络。只有 100 个图像包含来自 Fairchild 数据库的校准亮度值。==所有图像都包含线性 RGB 值==。用于评估的 50 个测试图像是从具有校准绝对亮度的 Fairchild 图像中随机选择的。用于训练的 LDR 内容是直接从数据集即时生成的，并通过如下所述的多种方式进行了增强。
### 1.5.2 数据增强

### 1.5.3 Optimization
使用 Adam 优化器，初始学习率为 7e-5，批量大小为 12，在前 10000 epoch 后，当损失达到稳定水平时，学习率就会降低 0.8 倍，直到学习率达到小于 1e-7。
在 NVIDIA P100 上训练总共花费了 130 个小时。
## 1.6 评价指标
Peak Signal to Noise Ratio（PSNR）:
Structural Similarity（SSIM）：
Multi-Scale Structural Similarity（MS-SSIM）:
HDR-VDP-2.2：
## 1.7 比较
- 四种 CNN 架构：
	- 第一个网络基于 UNet (UNT)，该架构在域间图像翻译任务中显示出强大的结果。
	- 第二个网络是首先用于着色的架构(COL)，它使用两个分支和一个类似于 ExpandNet 所用的融合层。
	- 这三个是用于 LDR 到 HDR 转换的网络架构(EIL) ，该方法的预测是使用作者在线提供的经过训练的网络创建的，应用于所有其他方法所用的同一测试数据集
- 传统方法：
	- LAN、BNT、AKY、REM、MAS、KOV、HUO、 HDR toolbox in Matlab 
# 2. 相关工作

## 2.1 LDR to HDR
- Expansion operators（EOs）：也称为逆或反向色调映射运算符，尝试从 LDR 内容生成 HDR 内容。
	- 全局方法：使用简单的函数在所有像素上均匀地扩展内容
	- 局部方法：通过结合使用分析函数和扩展图的方法
- Deep learning for Image Processing
	- 艾勒特森等人。使用类似 U-Net 的架构来预测曝光严重的内容的饱和区域的值，而不饱和区域通过应用逆相机响应曲线来线性化。远藤等人。
	- 使用改进的 U-Net 架构，该架构可以根据单次曝光预测多次曝光，然后使用标准合并算法生成 HDR 图像。


