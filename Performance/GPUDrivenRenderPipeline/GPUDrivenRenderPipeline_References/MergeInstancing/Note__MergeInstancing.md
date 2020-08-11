<!-- TOC -->

- [1. 论文课题](#1-论文课题)
- [2. Particle Trimming](#2-particle-trimming)
  - [2.1. ROP](#21-rop)
  - [2.2. 实现原理](#22-实现原理)
  - [2.3. work flow](#23-work-flow)
- [3. Merge-Instancing](#3-merge-instancing)
- [4. Phone-wire AA](#4-phone-wire-aa)

<!-- /TOC -->
[原文](./Persson_GraphicsGemsForGames.pptx)

# 1. 论文课题
- Particle Trimming
- MergeInstancing
- Phone-wire AA
- Second-Depth AA

# 2. Particle Trimming
当前的问题：
ALU、TEX、BW等硬件指标进步很快，ROP有进步但还是很慢。

## 2.1. ROP
ROP: Raster Operations Pipeline，对生成的像素进行AA等操作。

ROP的常用场景：
- 粒子
- 云
- Billboards
- GUI元素

现有解决方案：
- 绘制到低分辨率的render target上：减少需要处理的像素
- 滥用MSAA

本文的解决方案:
整理粒子的形状来减少浪费

## 2.2. 实现原理
原有的粒子等资源贴图存在大量alpha=0的区域会导致填充率的浪费，而且因为不会影响最终的结果，在很大程度上会照成overdraw，需要调整贴图形状来最小化浪费。

先说本文的实验结果：
<div align="center">

![][ParticleTrimmingResults]

</div>

## 2.3. work flow
- 设置alpha阈值
- 把每个阈值内的pixel加入凸包
  - 可以通过潜在角落测试来优化
- 减少凸包的顶点数量
  - 替换掉最不重要的边
  - 对所有的点重复上面的操作
- 暴力遍历所有的边排列组合
  - 选择最小的区域几何图形

# 3. Merge-Instancing
现状：
<div align="center">

![][MergeInstancingProblems]

</div>

work flow :

<div align="center">

![][MergeInstancingWorkFlow0]

![][MergeInstancingWorkFlow1]

</div>

> - instance data要配合freq
> - 必要时要添加辅助三角形

# 4. Phone-wire AA
虽然MSAA和MipMapping能解决大部分的AA问题，但是随着资源的精度上升、光照效果更真实等画质的提升，导致AA出现了更多需要解决的问题。这里介绍电话线一类物体的渲染AA解决方案。

原理：
当线的宽度要小于一个像素的时候，强制clamp宽度到1，并设置alpha进行混合。


[ParticleTrimmingResults]: ./ParticleTrimmingResults.jpg
[MergeInstancingProblems]: ./MergeInstancingProblems.jpg
[MergeInstancingWorkFlow0]: ./MergeInstancingWorkFlow0.jpg
[MergeInstancingWorkFlow1]: ./MergeInstancingWorkFlow1.jpg
