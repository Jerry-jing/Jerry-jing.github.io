---
title: Feature Guided Masked Autoencoder for Self-supervised Learning in Remote Sensing
author: Jing
date: 2024-05-17
---
# 1. 研究领域和主要问题
领域：遥感预训练（光学、SAR）
主要问题：光学遥感预训练、SAR 预训练
# 2. 团队
单位：不详
导师：Xiao Xiang Zhu, Fellow, IEEE
# 3. 前人做出的贡献
- 在遥感领域，用 MAE 这种方式进行预训练
	- 缺点：过分关注像素细节，限制模型的语义理解能力，特别是对于噪声 SAR 图像；对伪影和噪声敏感
# 4. 本文创新点
提出了 ==Feature Guided Masked Autoencoder (FG-MAE)==
![[FGMAE.png]]

使用 RS 图像特征替换重建目标
## 4.1 Target features
两类、四种类型的 RS 图像特征：
- spatially
	- CannyEdge：边缘检测算法
	- HOG：描述图像局部子区域内梯度方向分布的特征描述符
	- SIFT：用于从图像中提取独特且不变的局部特征
- spectrally
	- NDI including vegetation index, water index and built-up index：量化两个光谱带之间的差异来识别一种特面物体的技术，流行的 NDI 包括 NDVI、NDWI、NDBI
## 4.2 FGMAE-MS / SAR
- 多光谱
	- 重建多光谱的 Histograms of Oriented Graidents (HOG) 和 Normalized Difference Indices (NDI) ，将 HOG 和 NDI 结合，相互补充
- SAR
	- 重建 SAR 图像的 HOG，选择 HOG 是因为其计算效率和噪声鲁棒性
# 5. 实验
> 硬件设备、数据集是否很大、花费时间是否很多
- Self-supervised pretraining
	- Dataset：SSL4EO-S12 ( Sentinel-1 GRD、Sentinel-2 L1C )，全球 25 万个位置采样，每个地点有四个季节的四幅图像。多光谱 13 通道、SAR 图像 2 个通道（SAR 约 450G）
	- Data augmentation：RandomResizedCrop 调整为 224×224，使用 RandomHorizo​​ntalFLip 作为数据增强
	- Model architecture：采用 MAE 架构，仅 encoder 传输到下游任务
	- Optimization
		- batchsize：256
		- epoch：100
		- optimizer：AdamW
		- weight decay：0.05
		- basic learning rate：1.5e-4
		- warmed up：10
		- decay：consine schedule
	- 硬件：4 个 NVIDIA A100 GPU，多光谱 7 hours，SAR 4 hours
- Transfer learning 
	- scene classification
		- Dataset：EuroSAT (single-label land cover classification)、BigEarthNet-MM (multi-label land cover classification) 19 类标签、==零填充为 13 个通道== (12 通道)
		- 评估指标：OA (overall accuracy)、AA (class-wise average accuracy)
	- semantic segmentation
		- Dataset：DFC2020 (land cover segmentation) 10 级高分辨率分割标签
		- Model architecture：UperNet with ViT backbone
		- ==RSI-Segmentation library== for fine tuning、batchsize 8、40k iter、cross entropy loss、AdamW optimizer with layer decay、basic learning rate 1e-4、warm up 500 iter and then polynomial-decay
		- 评估指标：OA (overall accuracy)、AA (class-wise average accuracy)、mIoU (mean intersection over union)
# 6. discussion
SAR 去斑之后重建
