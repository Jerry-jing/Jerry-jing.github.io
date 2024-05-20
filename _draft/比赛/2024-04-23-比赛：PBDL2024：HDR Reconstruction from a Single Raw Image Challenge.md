---
title: HDR Reconstruction from a Single Raw Image Challenge
---
# 1. 赛题了解
## 1.1 任务 
HDR 重建，从单个原始图像重建 HDR 图像
![[sdr_hdr.png]]
> HDR（High Dynamic Range）：保留再现亮度。亮位更亮、暗位更深，色彩表现能力更好（远高于8 位精度）
> SDR（Standard Dynamic Range）：相较之下没有那么强的发光能力
> LDR（Low Dynamic Range）：（8 位精度）
> https://zhuanlan.zhihu.com/p/378840979
## 1.2 数据集
SRHDR 的 Raw-to-HDR，数据集包含成对的 LDR 和 HDR 图像
设备：佳能 5D Mark IV 相机
拍摄方式：包围曝光模式来捕捉同一场景的不同曝光
拍摄时间：在一天中不同时间捕获的（包括白天和黑夜）
照明条件：-3EV 到 +3EV，地面实况图是使用 HDR 合并方式创建（Debevec）
特点：高分辨率（6720 x 4480）、高位深（线性 HDR 格式、深度超过 20 位）、未经处理的原始传感器数据

train：300 对 6720 x 4480 的 .mat 文件（使用 `scipy.io` 访问），14 位 Raw 格式的输入图像、20 位 HDR 图像以及图像配置文件（白平衡和色彩校正矩阵）
Val：30 对 1024 x 1024 的 .mat 文件，格式与训练集一致
Test：30 个 1024 x 1024 的 .mat 原始图像

评估指标：峰值信噪比（PSNR）、色调映射（PSNR、PSNR-miu）、结构相似性（SSIM）和多尺度 SSIM（MS-SSIM）
![[evaluation_metric.png]]

LDR 输入是在具有挑战性的照明条件下捕获的，代表高动态范围场景的曝光过度和曝光不足的区域。数据集中相应的地面实况 HDR 图像是通过每个场景的包围曝光生成的，随后使用基本 HDR 融合算法进行合并（Debevec 等人，2008）
## 1.4 提交格式
`.zip` 文件
每个HDR 重建图像的命名与输入名称相同 `xx.mat`；没有子文件夹；
使用 `uint16` 是数据保存为 `mat` 格式

```
dict = {
	'__header__': header,
	'input': quantized_array, # 维度是 (长, 宽, 4)
	'wb' : wb,
	'pattern': pattern,
	'cam2rgb' : cam2rgb
}
```
# 2. 参考文献
https://openaccess.thecvf.com/content/ICCV2023/html/Zou_RawHDR_High_Dynamic_Range_Image_Reconstruction_from_a_Single_Raw_ICCV_2023_paper.html
## 2.1 常见的获取 HDR 数据的方式
- 从 `multi-exposure` 图像中重建
	- 缺点：有对齐问题，不对齐会造成鬼影效应（`ghosting effect`）
- 从 `novel camera sensors` 获取的图像中重建：HDR cameras、event cameras、infrared sensors
	- 问题：一般都是从商业相机中获取 HDR 数据（更具有商业价值）
- 从 `single-exposure` 图像中重建
	- 前人方式：从低比特的 `sRGB` 图像进行 `HDR` 重建
			- 缺点：得到 `sRGB` 的过程是有损的，不足以记录 `HDR` 场景中的细节。在高动态场景中经常会发生抖动。
			- `sRGB`：`sRGB` 图像经过相机的非线性化、裁剪、压缩和量化等处理，动态范围被限制在8比特，丢失了很多信息。（原始图像一般为 14 `bit`）
	- 本文方式：用原始图像进行 `HDR` 重建
		- 优点：原始图像具有更高的可用比特深度和更好的强度容忍度。
		- 大致方法：使用曝光掩膜（exposure mask）自适应的分离每个场景中过/欠拟合和光照良好的区域；设计了一个专门针对原始输入图像的深度神经网络，来利用硬区域（HDR 最难恢复的区域），提出了双强度引导（dual intensity guide），引导信息较少的图像通道和全局空间引导
	- 难点：欠曝光区域往往含噪，过曝光区域难以恢复

在这段文本中，作者提出了两种关键的引导机制来帮助从 Raw 图像重建 HDR 图像：
- 双强度引导（Dual intensity guidance）：基于 Raw 图像的通道可变属性（channel-variant attribute），即不同通道对光的敏感程度不同，绿色通道对光更敏感。因此，在欠曝区域，绿色通道的信号噪声比更高，可以用作其他通道（红色和蓝色通道）的引导信息。在过曝区域，由于红色和蓝色通道对光的敏感度较低，不容易过曝，因此可以用作过曝区域的引导信息。这样，双强度引导模块能够帮助恢复高动态场景中暗部和亮部区域的细节信息。
- 全局空间引导（Global spatial guidance）：考虑到在极端区域，即使是最有信息的通道也可能仍然不足以提供足够的细节。在这种情况下，可以利用更远区域的特征，因为可能存在相似的块可以帮助恢复这些困难区域。因此，作者提出了一个全局空间引导分支，使用一系列的 Transformer 块来提取特征，这些特征能够帮助重建困难区域。全局空间引导模块利用 Transformer 的自注意力机制，能够在更大的感受野内找到相似的块，从而引入更多的可用细节，并促进困难区域的重建。
## 2.2 创新点
### 2.2.1 提出 `learn an exposure mask`
- 目地： 将 `hard` 区域和 `easy` 区域分离，分离场景中过曝光、欠曝光和光照良好的区域 
- 具体方式：使用深度神经网络来进行曝光掩膜
	- 网络结构：
	- 损失：
### 2.2.2 提出一个 `deep network`
- 目地：处理 `hard regions`
- 具体方式：双强度引导
	- 绿色通道具有更高的强度值，不同的通道共享相似的边缘；绿色通道对光更敏感，曝光不足的区域具有更高的信噪比（SNR）、红色和蓝色过曝光区域可能性较小
	- Bayer pattern： $H×W$ 到 $H/2×W/2×4$
	- 
### 2.2.3 直接从单个 `Raw` 重建 `HDR`
- 可能集成到现代相机处理流水线中


## 2.3 相关的方法
> [github 整理仓库](https://github.com/rebeccaeexu/Awesome-High-Dynamic-Range-Imaging
###  2.3.1HDRCNN
- 有代码

- ExpandNet
- DeepHDR
- HDRUNet

[CodaLab - 竞赛 (upsaclay.fr)](https://codalab.lisn.upsaclay.fr/competitions/17839)

[RAW、RGB和YUV格式_raw数据-CSDN博客](https://blog.csdn.net/weixin_42463482/article/details/127702176)
其中有 HdM HDR 数据集的格式：
[CVPR21 《A Two-stage Deep Network for High Dynamic Range Image Reconstruction》HDR高动态范围图像重建论文复现-CSDN博客](https://blog.csdn.net/qq_41588833/article/details/119774573)
