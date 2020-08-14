<!-- TOC -->

- [1. 调研](#1-调研)
- [2. 原理](#2-原理)
  - [2.1. 阴影遮罩](#21-阴影遮罩)
  - [2.2. ShadowCaster剔除](#22-shadowcaster剔除)

<!-- /TOC -->

[原文](./Chasing%20Shadows.pdf)

# 1. 调研
可看做是级联和[ShadowCasterCulling](./../Practical,DynamicVisibilityForGames/Note_Practical,DynamicVisibilityForGames.md)的一种变体。

具体的详见原文。

# 2. 原理
## 2.1. 阴影遮罩
如果阴影像素最终被采样了，那么这样的像素就被称作有用的像素。这些有用像素的合集就是阴影遮罩。

最直接的生成方式：
通过把depth buffer的点重新投影到光源空间，然后在光源空间内光栅化。

缺点：
- 全分辨率情况下，会导致工作量巨大
- 原有几何体的拓扑关系会丢失。如果有要考虑投影点云的连接性的操作，基本做不到。
- 这种方式生成的遮罩有可能产生小洞等，会导致错误的剔除和丢失的阴影。

本文的处理方式：
- 把depth buffer划分为屏幕空间的多个tile grid
- 通过tile中的最近最远距离，确定世界空间中一个紧凑的包围视锥
  - 通过min/max depth pyramids来构建紧凑的包围视锥
  - 通过给定的全分辨率depth buffer，采用缩小采样的方式直到得到tile grid的分辨率，就能计算得到一个min/max depth pyramid
  - 保证每个level的map都是长宽都是$2^N$
  - 生成过程中要排除地形和天空盒的影响，也就是depth=1时要谨慎处理，只考虑depth小于1的情况
- 光栅化各tile grid
  - 直接通过几何着色器进行操作，通过深度查找使用哪个depth pyramid然后进行光栅化操作
  - 关闭depth和color buffer，用stencil记录信息

<div align="center">

![][CSWorkFlow]

</div>

## 2.2. ShadowCaster剔除
两个pass：
- 根据灯光视锥和shadow mask进行剔除
- 绘制shadow map


[CSWorkFlow]: ./CSWorkFlow.jpg