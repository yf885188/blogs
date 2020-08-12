<!-- TOC -->

- [1. 原理](#1-原理)
  - [1.1. 结构](#11-结构)
  - [1.2. work flow](#12-work-flow)
- [2. 应用](#2-应用)
  - [2.1. 遮挡剔除](#21-遮挡剔除)
  - [2.2. 粒子剔除](#22-粒子剔除)
  - [2.3. 体积阴影clamp](#23-体积阴影clamp)

<!-- /TOC -->

[原文](./N-Buffers%20for%20efficient%20depth%20map%20query.pdf)

# 1. 原理
## 1.1. 结构
- 跟Hi-Z buffer不同的是，这里的每层depth buffer尺寸都一样。
- 0级是一个标准的深度图
- i级的像素值是0级中以这个像素点为左下角起点，ixi尺寸大小内最大的Z值。

<div align="center">

![][N-BuffersStructure]

</div>

## 1.2. work flow
构建：
$$l_{i+1} = max(max(l_i[x,y], l_i[x, y+2^i]), max(l_i[x+2^i, y], l_i[x+2^i, y+2^i]))$$

query:
- 跟Hi-Z一样，通过屏幕空间AABB来确定等级，$l = ceil(log_2(s_aabb.width, s_aabb.height))$
- 到底是按照长边用4个quad覆盖求，还是按照短边用1个quad覆盖求。也用实验的方式求处理两种方式下浪费的像素值。结论是用4个quad求的话，准确率要高些。

# 2. 应用
## 2.1. 遮挡剔除
主要过程与Hi-Z差不多，可以参考[Greene93](./../Hi-Z/Note__Hi-Z.md)和[Hill11](./../Practical,DynamicVisibilityForGames/Note_Practical,DynamicVisibilityForGames.md)

## 2.2. 粒子剔除
一般的粒子系统都是通过点精灵实现，传给GPU的只有一个坐标点表示中心点位置。为了使用N-Buffer可以把粒子扩展成四边形。

## 2.3. 体积阴影clamp
也可以参考[Hill11](./../Practical,DynamicVisibilityForGames/Note_Practical,DynamicVisibilityForGames.md)

<div align="center">

![][ShadowVolumeCulling]

</div>


[N-BuffersStructure]: ./N-BuffersStructure.jpg
[ShadowVolumeCulling]: ./ShadowVolumeCulling.jpg