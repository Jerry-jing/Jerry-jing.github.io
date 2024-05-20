2024-04-13 -> 2024-04-20
- [x] 修改 dataset，读取 .mat
- [x] 修改 model，可以读入 4 通道
2024-04-20 -> 2024-04-30
- [x] 尝试不同的方法进行 train
- [x] 涨点！
# 1. 调研
## 1.1 深度学习方法

| 方法                       | 链接                                                                                                                                                                                                                                              | 问题                          | 是否可以使用 | 尝试             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | ------ | -------------- |
| HDRCNN                   | [gabrieleilertsen/hdrcnn：使用深度 CNN 从单次曝光中重建 HDR 图像 (github.com)](https://github.com/gabrieleilertsen/hdrcnn)                                                                                                                                     | 代码很少                        | 是      | tensorflow     |
| DrTMO_unofficial_pytorch | [shleecs/DrTMO_unofficial_pytorch: Unofficial pytorch Version of Deep Reverse Tone Mapping (SIGGRAPH ASIA'2017) (github.com)](https://github.com/shleecs/DrTMO_unofficial_pytorch/tree/master)                                                  | 时间很老，代码很少                   | 是      | 没有 model.py 文件 |
| ExpandNet                | [dmarnerides/hdr-expandnet：ExpandNet 的训练和推理代码 (github.com)](https://github.com/dmarnerides/hdr-expandnet)                                                                                                                                       | 时间很老，代码很少                   | 是      | 尝试了，效果不错       |
| FHDR                     | [mukulkhanna/FHDR：全球 SIP 2019 论文“FHDR：使用反馈网络从单个 LDR 图像重建 HDR 图像”的 PyTorch 实施 (github.com)](https://github.com/mukulkhanna/fhdr)                                                                                                                 | 输入：jpg/png<br>输出：exr        | 是      | 尝试了效果还行        |
| SingleHDR                | [alex04072000/SingleHDR: [CVPR 2020] Single-Image HDR Reconstruction by Learning to Reverse the Camera Pipeline (github.com)](https://github.com/alex04072000/SingleHDR)                                                                        | tensorflow 格式的，比较难改         | 是      |                |
| twostageHDR_NTIRE21      | [twostageHDR_NTIRE21](https://github.com/sharif-apu/twostageHDR_NTIRE21)                                                                                                                                                                        | NTIRE21 HDR 挑战（单帧）          | 是      |                |
| Deep Recursive HDRI      | [李思英/Deep_Recursive_HDRI (github.com)](https://github.com/Siyeong-Lee/Deep_Recursive_HDRI)                                                                                                                                                      | 多曝光                         | 否      |                |
| GlowGAN                  | [[2211.12352] GlowGAN: Unsupervised Learning of HDR Images from LDR Images in the Wild (arxiv.org)](https://arxiv.org/abs/2211.12352)                                                                                                           | 没有代码                        | 否      |                |
| DiffHDRsyn               | [DiffHDRsyn](https://github.com/JungHeeKim29/DiffHDRsyn/tree/main?tab=readme-ov-file#diffhdrsyn)                                                                                                                                                | 使用的 VDS 这个数据集是多曝光的          | 否      |                |
| RawHDR                   | 无                                                                                                                                                                                                                                               | 没有代码                        | 否      |                |
| Deep-HdrReconstruction   | [marcelsan/Deep-HdrReconstruction: Official PyTorch implementation of "Single Image HDR Reconstruction Using a CNN with Masked Features and Perceptual Loss" (SIGGRAPH 2020) (github.com)](https://github.com/marcelsan/Deep-HdrReconstruction) | 没有 train，自己推理可视化效果其实还可以     | 否      |                |
| JSI-GAN                  | [JihyongOh/JSI-GAN：[AAAI 2020] JSI-GAN 的官方存储库。 (github.com)](https://github.com/JihyongOh/JSI-GAN/tree/master)                                                                                                                                  | 是视频 SDR-> HDR，但是不直接使用时序帧的信息 | 可能可以   |                |
| HDRUnet                  | [chxy95/HDRUNet: CVPR2021 Workshop - HDRUNet: Single Image HDR Reconstruction with Denoising and Dequantization. (github.com)](https://github.com/chxy95/HDRUNet/tree/main)                                                                     |                             | 是      | 尝试过分数不高        |
| ResNet                   |                                                                                                                                                                                                                                                 |                             |        |                |
|                          |                                                                                                                                                                                                                                                 |                             |        |                |
## 1.2 普通方法

## 1.3 数据集

| 数据集名称  | 主要内容                                                                                             | 论文                                                                                                                         | 地址                                                                                    |
| ------ | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| SI-HDR | 包含 181 张 HDR 图像组成。<br>每张图像包括：1） 一个 RAW 曝光堆栈，2） 一张 HDR 图像，3） 两种不同曝光下的模拟相机图像 4） 6 种单图像 HDR 重建方法的结果 | [计算机实验室 – 项目：单图像 HDR 重建方法的比较 — 质量评估的注意事项 (cam.ac.uk)](https://www.cl.cam.ac.uk/research/rainbow/projects/sihdr_benchmark/) | [SI-HDR 数据集 \|带代码的论文 (paperswithcode.com)](https://paperswithcode.com/dataset/si-hdr) |
|        |                                                                                                  |                                                                                                                            |                                                                                       |


## 1.3 使用的方法及结果
- val 阶段

| 方法        | 方式                   | Score | PSNR  | SSIM | PSNR-μ | MS-SSIM | 链接                                                        | epoch |
| --------- | -------------------- | ----- | ----- | ---- | ------ | ------- | :-------------------------------------------------------- | ----- |
| 原图        | 提交格式量化为 float32      | 23.53 | 14.10 | 0.71 | 13.37  | 0.74    |                                                           |       |
| HDRUNet   | 自己训练，提交格式量化为 float32 | 28.41 | 14.69 | 0.77 | 16.58  | 0.82    | [HDRUNet](https://github.com/chxy95/HDRUNet==)            |       |
| ExpandNet | 自己训练，提交格式量化为 float32 | 54.72 | 27.54 | 0.92 | 28.03  | 0.95    | [ExpandNet](https://github.com/dmarnerides/hdr-expandnet) | 750   |
|           |                      |       |       |      |        |         |                                                           |       |
- test 阶段

|          方法          |      epoch       | Score | PSNR  | SSIM | PSNR-μ | MS-SSIM |             备注             |
| :------------------: | :--------------: | :---: | :---: | :--: | :----: | :-----: | :------------------------: |
|      ExpandNet       |       650        | 53.75 | 27.15 | 0.88 | 27.84  |  0.92   |                            |
|      ExpandNet       |       750        | 54.17 | 27.19 | 0.88 | 28.26  |  0.92   |                            |
|      ExpandNet       |       850        |       |       |      |        |         |                            |
|      ExpandNet       |        融合        | 55.29 | 27.69 | 0.88 | 28.82  |  0.92   | 450、550、650、750、850、950 融合 |
|         FHDR         |       700        | 46.82 | 25.98 | 0.80 | 23.01  |  0.87   |                            |
|                      |                  |       |       |      |        |         |                            |
|        KUNet         | 900(135000 iter) | 30.54 | 16.85 | 0.72 | 17.67  |  0.77   |                            |
|                      |      46000       | 31.26 | 16.84 | 0.74 | 17.73  |  0.80   |                            |
| ExpandNet 与 KUNet 融合 |                  | 41.06 | 22.0  | 0.78 | 21.53  |  0.85   |                            |
|        RECNet        |                  |       |       |      |        |         |                            |
|       DeepHDR        |                  |       |       |      |        |         |                            |
|        HDRCnn        |                  |       |       |      |        |         |                            |
过拟合可能会有用
希望尝试数据增强、修改 Loss 的方法

KUNet 对暗处重建比较好；FHDR 纹理比较清晰，但色彩不好（整体感觉不如其他效果好）；ExpandNet 整体效果好，但是整体比较暗。

# 其他收获

| 方法                                                                                       | 公司  | 问题                                  | 领域     | 发表期刊      | 其他              |
| ---------------------------------------------------------------------------------------- | --- | ----------------------------------- | ------ | --------- | --------------- |
| HDRTVNet                                                                                 | 快手  | SDRTV -> HDRTV                      | 图像视频处理 | ICCV 2021 | 扩展版本 HDRTVNet++ |
| HDRTVDM                                                                                  |     | SDR 到 HDRTV、ITM（反色调映射）或 HDR/WCG 上转换 |        |           |                 |
| Learning a Practical SDR-to-HDRTV Up-conversion using New Dataset and Degradation Models |     | HDR-to-SDR degradation              |        |           |                 |

# 问题
HDR 重建和图像增强感觉效果上并没有太大差别。
雪健推荐的低光增强的代码：[albrateanu/LYT-Net： LYT-Net：基于YUV转换器的轻量级网络，用于低光图像增强 (github.com)](https://github.com/albrateanu/LYT-Net)

HDR 重建和超分之间的关系。