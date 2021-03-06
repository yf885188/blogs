<!-- TOC -->

- [1. SDSM(SampleDistributionShadowMaps)简介](#1-sdsmsampledistributionshadowmaps简介)
- [2. 调研](#2-调研)
- [3. 原理](#3-原理)
  - [3.1. 深度分区](#31-深度分区)
  - [3.2. 深度分区的变种算法](#32-深度分区的变种算法)
  - [3.3. 获取紧凑的视锥](#33-获取紧凑的视锥)
- [4. 不足](#4-不足)
  - [4.1. 剔除问题](#41-剔除问题)
  - [4.2. 阴影闪烁问题](#42-阴影闪烁问题)

<!-- /TOC -->

[原文](./Sample_distribution_Shadow_Maps.pdf)

# 1. SDSM(SampleDistributionShadowMaps)简介
- 级联阴影（阴影Z分区的传统做法）需要手动或者经验去调，花费大量时间并且效果只在某些视点好
- SDSM每帧根据阴影分辨率做优化，还保持固定的内存占用和表现。并通过分析当前视点的阴影采样分布，来完成紧紧约束的，视点独立的阴影Z分区。

本文达成的目标：
- 通过使用者的采样分布来自动进行相机视锥分区的方法
- 通过采样分布计算能紧紧包裹灯光空间分区视锥从而大幅提高阴影质量的算法

# 2. 调研
具体参看原文

# 3. 原理

还可以参考[blog](./http://www.klayge.org/2013/05/07/%E5%A4%A7%E8%8C%83%E5%9B%B4shadow-map%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9Asdsm/)

两个pass：
- 计算场景中需要采样的深度范围分布
- 使用深度分区和采样分布计算在灯光空间能紧紧包裹采样区域的分区视锥

## 3.1. 深度分区
原则：尽量让分区有最小的far-to-near比率，这样从整个视锥角度上来看，分辨率的影响就会变小。
算法： the basic logarithmic partitioning，[min/max reduction过滤](https://docs.microsoft.com/zh-cn/windows/win32/direct3d11/tiled-resources-texture-sampling-features?redirectedfrom=MSDN#minmax-reduction-filtering)

work flow:
- 渲染一遍场景，得到depth texture
- 根据depth texture,获取最大最小depth
- 根据depth的范围根据log的距离分布平行切成几个区域

<div align="center">

![][SampleDistribution]

</div>

## 3.2. 深度分区的变种算法
- 自适应logarithmic partitioning：太费算力
- K-means：效果好坏跟定义的权值有关，也即不能保证完全划分合理，还比之前的算法更费算力

## 3.3. 获取紧凑的视锥
- 采样的时候已经把遮挡剔除等因素考虑到了，然后根据采样的点，能获得一个紧凑的视锥

# 4. 不足
## 4.1. 剔除问题
整个过程都是独立在GPU上完成的，这样CPU就不能参与进来针对各个分区做剔除，只能保守点儿，把只要跟整体视锥相交的物体都绘制了。

对于很大很复杂的场景，解决这个问题的最好方案是让CPU和GPU保持同步，CPU 阻塞等待GPU处理完成后read back。这种方式的影响比传统方式带来的影响要小，SDSM生成的视锥更紧凑，带来的效果也填补了这种损失和其他所有SDSM的负载影响，甚至在一些中度复杂的场景中也是如此。

对于GPU来说，比较理想的解决方式是视锥直接剔除，然后提交结果直接渲染。

## 4.2. 阴影闪烁问题
旋转的时候，分区阴影会闪烁。最直观的做法是保证每次分区的位置都是一样的，但是这样等于不然改变视锥。

比较理想的做法是实现亚像素阴影分辨率，跟MSAA类似。

[SampleDistribution]: ./SampleDistribution.jpg