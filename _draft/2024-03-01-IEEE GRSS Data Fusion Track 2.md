---
layout: default
title: IEEE GRSS Data Fusion Contest Track 2
---

> 比赛官网：[Image Analysis and Data Fusion - GRSS-IEEE](https://www.grss-ieee.org/technical-committees/image-analysis-and-data-fusion/?tab=data-fusion-contest)
> 任务：二类的语义分割，赛道一使用SAR数据，赛道二使用光学数据。

# 一、数据处理

## 1. 数据介绍

the water surface from Copernicus Sentinel-2 and Landsat optical imageries.

<img title="" src="file:///D:/workplace/Jerry-jing.github.io/_posts/img/Multispectral_img.png" alt="Multispectral_img.png" data-align="center" width="283">

遥感卫星不同光谱带介绍（Band 1 ～ Band 13）：[遥感卫星不同光谱带介绍(Band 1 - Band 13)_coastal aerosol-CSDN博客](https://blog.csdn.net/kuangxinyaya/article/details/123059314)

- Coastal aerosol（443nm）：沿海气溶胶带在沿海、测深和气溶胶研究中特别有用。
  
  特点：具有穿透水的能力（在清澈的水中高达20～30米）
  
  由于沿海气溶胶带对云、烟雾和雾气更敏感，因此在图像处理中**用于过滤云**。地球观测者正在沿海气溶胶带的帮助下微调无云正交成像作为基础地图。

- Blue（490nm）：深水成像、对大气雾霾最敏感（用于检测烟雾）、将云与雪和岩石分开。

- Green（560nm）：植物活力和植被、藻类和氰化物。

- Red（660nm）：热带土壤、建筑环境和地质特性

- Near Infrared large band（840nm）：健康植被进行分类、水和植被更容易分开

- SWIR-1短波红外线1（1600nm）：区分干土和湿土，**具有很好的穿透云层的能力**

- SWIR-2短波红外线2（2200nm）：用于成像土壤类型、地质特征以及铜和硫酸盐等矿物，水的吸收能力要强得多

- Quality assessment：**对遥感图像进行像素级评估，可能是有云层遮挡效果差一些，没有云层遮挡效果好一些**

- Merit DEM：地形高度数据；

- Copernicus DEM：地形高度数据；

- ESA world cover map：详细的地表覆盖数据（11分类），初始分类精度较低；

- Water occurence probability：水体出现概率的数据

- Binary water mask：结果掩膜

## 2. 数据处理

### 最大最小归一化

统计遥感图像 tif 数据的时候，发现其为 uint16 格式的数据，有很大的数据，需要将其归一化为 uint8 ，这里使用的是最大最小归一化方法。

> 师兄的经验：不同的库（如 opencv、pillow、tifffile等）保存和读取的方式不同，用什么库保存，就用什么库去读取。

```python
scaled_data = ((option_image - option_image.min()) / (option_image.max() \ 
 - option_image.min()) * 255).astype('uint8')
```

### 将 channel 切片

因为多光谱数据波段较多，可能会出现信息冗余，而且不同波段对于水的特性不同，加之官方的 baseline 中就将不同的波段切片，所以将多光谱图像不同波段的信息分开。

```python
import os.path

import cv2
import rasterio
import numpy as np
import matplotlib.pyplot as plt

root = ''
save_root = ''
channel_name = ['coastal_aerosol', 'blue', 'green', 'red', 'near_infrared', 'swir1', 'swir2', 'QA', 'merit_dem',
           'copernicus_dem', 'esa_world_cover_map', 'water_occurence_probability']
for i in range(306, 431):
    image_name = str(i) + '.tif'
    image_path = os.path.join(root, image_name)

    with rasterio.open(image_path) as src:
        image = src.read()
        for i in range(image.shape[0]):
            channel = image[i]
            print(i, np.unique(channel, return_counts=True), channel.shape)

            normalized_image = (channel - channel.min()) / (channel.max() - channel.min()) * 255
            normalized_image = normalized_image.astype('uint8')

            save_path = os.path.join(save_root, channel_name[i])
            os.makedirs(save_path, exist_ok=True)

            cv2.imwrite(os.path.join(save_path, image_name), normalized_image)
```

### 计算数据的 mean 和 std

在数据读入网络之前，有一个对数据进行标准化的处理操作，默认给的值可能不适用于数据的特点，需要自己计算每个通道的 mean 和 std。

### 数据切分操作

因为数据本身没有划分验证集，为了节省在网站上提交的次数，需要在本地划分验证集，使用的是  train ： val = 8 ： 2 。

## 3. 消融实验

> 做数据的消融实验，看每个波段对水的特性更强。

小强师兄已经做了不同波段对水的特性（小强师兄太强了，一个人跑了一个实验室的量），具体的方法是：用同一个模型同样的配置，更换不同的波段的 tif 图片，这里用到了 mmseg 框架，在第 100 个 iter 的时候测试，比较不同波段的得分。

<img src="file:///D:/workplace/Jerry-jing.github.io/_posts/img/Data_ablation_experiments.png" title="" alt="Data_ablation_experiments.png" data-align="center">

这里师兄得到的分数较高的波段是 NIR、SWIR1、SWIR2，但是后续用这三个波段跑到时候效果没有 Red、NIR、SWIR1 波段跑的分数高，最后使用了 Red、NIR、SWIR1。

后面找到了一个博客介绍 LandSat8 不同波段组合的作用：[遥感数字图像处理（实验二）——假彩色合成与伪彩色合成（密度分割）-CSDN博客](https://blog.csdn.net/qq_50559644/article/details/123900682)。这里的  5 、 6 、 4 波段就是 NIR 、 SWIR1 、Red 三个波段。

<img src="file:///D:/workplace/Jerry-jing.github.io/_posts/img/LandSata8不同波段作用.png" title="" alt="LandSata8不同波段作用.png" data-align="center">

下面的是得到 Red 、NIR 、SWIR1 三个波段假彩色图的代码。

```python
import os
import cv2
import rasterio
import numpy as np

raw_image_path = ''
save_path = ''
os.makedirs(save_path, exist_ok=True)

for image_name in os.listdir(raw_image_path):
    image_path = os.path.join(raw_image_path, image_name)
    with rasterio.open(image_path) as src:
        sar_image = src.read()

        red = sar_image[3]
        nir = sar_image[4]
        swir1 = sar_image[5]
        option_image = np.stack((red, nir, swir1), axis=0)

        # 将栅格数据的范围缩放到0-255之间（可选）
        scaled_data = ((option_image - option_image.min()) / (option_image.max() - option_image.min()) * 255).astype('uint8')
        scaled_data = scaled_data.transpose(1, 2, 0)

        cv2.imwrite(os.path.join(save_path, image_name), scaled_data)
        print("success")
```

# 二、模型选择

> 师兄给了一个 baseline ，在 mmseg 中， 使用 knet + resnest101 + fcn 模型，跑 10 k 个 iter（大约 500 个 epoch ），加上 tta， 已经到 0.964 了！（之前搞了七八个模型硬投票不如师兄一个模型好）

师兄讲了他挑选模型的准则：因为任务比较简单，所以挑选模型大概选一些轻量化的模型，不选择那些比较大的模型。

现阶段跑通了 knet + resnest101 + fcn， 加了 tta，在 test 上的分数已经 0.964 了。

## 1. 调 batch_size

根据自己的显卡调整 batch_size

## 2. 调 lr 和学习策略

根据自己的 batch_size 和 要跑多少个 iter 调整 lr 和学习策略（需要进一步学习，有时候感觉很玄学）

## 3. 调具体要训练多少个 iter

按照 8 ： 2 划分验证集，之后确定最佳的 iter 的位置

# 三、模型优化

## 1. 特征融合的三种方式

- 在数据输入端融合：进入模型之前进行融合，使用 mmseg 中的一个读取数据的pipeline：LoadMutilpleRSImageFromFile，可以读取多张图片。
  
  - 融合 Red、NIR、SWIR1 和 CDEM
  
  - 融合 Red、NIR、SWIR1 和其他

- 在特征端进行融合
  
  - 比较简单的思路：不同的图片通过 backbone 得到多尺度的特征图，之后对应融合起来（加、或者拼接之后卷积降维）
  
  - 进阶版思路：多模态分割（参考论文）

- 在输出端融合：软投票、硬投票、加权硬投票

## 2. 使用半监督的方法

用现有的模型生成伪标签，但前提是监督模型做到最好。  

# 四、一些技巧

## 1. 使用 tta

[深度学习竞赛中常见的一种手段：测试时增强（TTA）-云社区-华为云 (huaweicloud.com)](https://bbs.huaweicloud.com/blogs/detail/193404)

## 2. 多个模型进行投票

[【机器学习之模型融合】Voting投票法基础理论_加权投票-CSDN博客](https://blog.csdn.net/weixin_46803857/article/details/128685303)

软投票、硬投票、加权硬投票

# 五、一些遇到的问题

## 1. mmseg 中输入图像大小

输入图像大小之前设置为 128 ，每个模型的效果都比其他同学的低几个点，之后问了师兄，应该是图像输入太小之后最开始几次卷积会损失很多信息。

## 2. 调整 batch_size 之后进一步调 lr 和学习策略

之前调整 batch_size 但是没有进一步调整 lr 和学习策略，所以导致分数很低。

# 六、需要进一步了解的问题

1. 为什么图片的水域和 label 中出现的水域明显不同？图片中的水域很小，label 中的水域很大

2. swin 用什么大小，用什么窗口大小？

3. mIoU、aAcc、mAcc、mFscore之间的差别

4. mmseg 中 param_scheduler 对结果有多大影响是不是 PloyLR 结尾之后学习率就是 0 [pytorch动态调整学习率之Poly策略_poly学习率-CSDN博客](https://blog.csdn.net/guzhao9901/article/details/114373603)

5. sklearn 包里面有好东西

6. 如何评价模型到底大不大，就比如说师兄这里说的任务简单，模型就简单一些。

7. 模型的 pth 文件到底保存了什么。resume 和 只加载模型权重

8. Fcn 、deeplabv3 、 pspnet 、 upernet 、 SegNet

9. 模型的 Flops、FPS、Params、Memory footprint、Energy Efficiency、Computation Efficiency 分别是什么意思、单位是什么、能用来表征什么

10. 显存占用什么决定？
