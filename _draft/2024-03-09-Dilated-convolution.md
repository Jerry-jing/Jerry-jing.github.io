```yaml
layout: default
title: Dilated Convolution
```

> 别名：空洞卷积、膨胀卷积
> 
> 作用：增加感受野；保证原输入特征图 W、H

# 一、Dilated Convolution

膨胀因子 / 膨胀系数：r 表示相邻参数之间的距离

为什么使用膨胀卷积？既能增加感受野，也能保证输入输出特征图的 W、H 不会发生变化。

不使用最大池化下采样，会导致特征图对应原图的感受野变小。

gradding effect 的问题：尽量让高层上的任意一个 pixel ，使用到底层的数据是一个连续的区域

# 二、Understanding Convolution for Semantic Segmentation

Hybrid Dilated Convolution （HDC）: 连续使用多个膨胀卷积时，如何设计它的一个膨胀系数
