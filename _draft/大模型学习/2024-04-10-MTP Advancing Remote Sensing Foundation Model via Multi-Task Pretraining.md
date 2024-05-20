---
title: "MTP: Advancing Remote Sensing Foundation Model via Multi-Task Pretraining"
author: Jing
date: 2024-04-10
tags:
  - 大模型
---
> Multi-Task Pretraining
> the Multi-Task Pretraining (MTP) paradigm

[code](https://github.com/ViTAE-Transformer/MTP)
[paper]([https://arxiv.org/pdf/2403.13430.pdf](https://arxiv.org/pdf/2403.13430.pdf))
# 1. 研究领域和主要问题
遥感预训练
# 2. 团队
武汉大学 张良培团队
# 3. 前人做出的贡献、related work
## 3.1 前人贡献
- ImageNet 预训练权重被用于 RS 任务中
	- 缺点：RS 图像是从鸟瞰角度捕获的，缺乏自然图像的鲜艳色彩，而且空间分辨率较低，这些问题可能使得模型微调功能受阻。
	- 导致 RS 领域必须有自己的大模型
- 遥感领域大的数据集的产生
	- RS 场景标记数据集：fMoW、BigEarthNet、MillionAID（光学遥感）
	- 缺点：相较于 ImageNet 等自然图像数据集还是偏少，而且标记 RS 图像需要相关专业知识、大量时间和劳动成本，大量 RS 数据集没有标注，需要发展无监督或者半监督
- self-supervised learning（SSL）
	- contrastive-based learning：使相似样本更接近、不同样本远离。*然而对于训练大规模模型而言，遥感数据 design pretext task 和 gather requisite data 效率很低。*
	- generative-based learning：代表是 masked image modeling（MIM）
	- 结合 contrastive-based learning 和 generative-based learning
- 结合多源数据进行预训练
	- [45]teacher-student framework：同时集成了 ImageNet 监督预训练和 RS 无监督预训练。
	- [46]采用 ImageNet 的表示来增强 MIM 的学习过程，以改进 RS 基础模型。
	- [12]和[11]分别使用对比自监督学习或 MAE 在自然图像和 RS 图像上顺序预训练模型
- RS 数据集缺乏 segmentation 和 rotated object detection 的标签
	- SAMRS：通过 SAM 从现有 RS rotated object detection datasets 中得到的大规模分割数据集
## 3.2 related work
- 有监督学习 RS 基础模型
- 多阶段预训练 RS 基础模型
	- 遥感图像和自然图像的 domain gap
	- [11]设计了一种顺序预训练方法，现在 ImageNet 上，后在目标 RS 数据集上，进行 MIM 预训练
	- [12]提出了一种受类人学习启发的策略，首先对自然图像执行对比 SSL，然后冻结浅层权重并在 RS 数据集上执行 SSL
	-  [61]通过将辅助数据和最终任务目标集成到学习过程中，为 NLP 任务引入了更强的最终任务感知训练
	- [13]使用通用分割器（例如，UperNet [62]和Mask2Former [63]）和SAMRS数据集引入了额外的分割预训练，提高了RS分割任务中的模型准确性
- 多任务预训练 RS 基础模型
	-  [64]引入了多任务 SSL 表示学习，结合图像修复、变换预测和对比度学习来提高RS图像的语义分割性能，但是它仅限于语义分割任务上微调预训练模型。
	-  [59]通过将 Swin-Base 与现有网络的七个头（例如 Faster-RCNN 和 UNet）集成，设计了一个多任务模型，促进对多任务注释的 Satlas 数据集的训练。然而，他们的方法缺乏典型 RS 旋转对象任务的结合，只专注于 RS 分类数据集。
# 4. 本文动机、创新点和主要贡献
## 4.1 动机
- 前提：预训练和微调之间的任务差异，[5]强调预训练和微调任务之间表示粒度不匹配的影响。在场景级分类任务上预训练的模型在类似任务上进行微调时表现良好，但在像素级分割任务上却表现不佳。
- 问题：**是否可以通过结合具有不同表示粒度的多个任务的额外预训练来显著增强 RS 基础模型的表示能力？**
- 研究：多任务预训练范式（MTP），弥合上游和下游任务之间的差距，获得强大的 RS 基础模型；应用在 RS、自然图像上
## 4.2 创新点
- 引入stage-wise multi-task pretraining approach （分阶段多任务预训练）增强 RS 基础模型，解决上游预训练和下游微调任务之间的差异
## 4.3 主要贡献
- 利用 MTP 在 SAMRS 数据集上预训练超过 300M 参数的具有代表性的 CNN 和 Vision Transformer foundation models
- MTP 显著提高了 RS 基础模型的表示能力，在 scene classification、semantic segmentation、object detection 和 change detection 等下游任务中取得了卓越的性能。
# 5. 具体细节、实验
![[MTP.png]]
## 5.1 SAMRS Dataset
定义：大型遥感分割数据集
包含：来自SOTA、SIOR 和 FAST三个数据集的 105,090 张图像和 1,668,241 个实例
来源：*通过 SAM 转换边界框，将大规模 RS 目标检测数据集 DOTA-V2.0、DIOR、FAIR1M-2.0 转换而来*
## 5.2 Backbone Network
采用 RVSA 和 InterImage 作为代表性的 vision transformer-based and CNN-based foundation models
## 5.3 Implementation of Multi-Task Pretraining
三种模型检验 MTP：ViT-B + RVSA、ViT-L + RVSA、InternImage - XL
*这里有很多细节还不是很清楚，需要再看一看*
- overalll loss for MTP：
$L=L_{rod} + L_{ins_b} + L_{ins_m} + L_{sem}$
- SAMRA 包含三个 sets
$L=$
## 5.4 实践中
整个框架基于 MMSegmentation， MMDetection，MMRotate
## 5.5 Experiments
- 整个训练花费很长时间和很多卡（TABLE Ⅱ）
- Scene Classification
	- 使用数据集：EuroSAT、RESISC-45
- Horizontal Object Detection
- Rotated Object Detection
- Semantic Segementation
- Change Detection
- Further Investigations and Analyses

预训练针对 scene classification、horizontal and rotated object detection、semantic segmentation、change detection 任务进行微调。

## 5.6 代码可以借鉴的地方
- Dataset：有三个 Dataset，也是用这种持续学习的方式（Scale-MAE 的那种3、4、8通道持续学习也用的这种）
- Model：一个 decoder 和 三个 decoder，将所有任务得到的 loss 相加
	- 一个 forward 里面要做三个任务 ？
	- 一个 forward 里面一个任务，持续进行？
# 6. 达到的效果