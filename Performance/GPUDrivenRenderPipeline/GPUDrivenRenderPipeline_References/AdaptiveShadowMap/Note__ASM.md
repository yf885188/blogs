<!-- TOC -->

- [1. 调研](#1-调研)
- [2. 原理](#2-原理)
  - [2.1. 创建节点](#21-创建节点)
  - [2.2. 删除节点](#22-删除节点)

<!-- /TOC -->

[原文](./ASM.pdf)

# 1. 调研
详细看原文

# 2. 原理
思路：
- 不需要统一的高分辨率
- 只有包含阴影边界的需要密集采样
- 在这些区域才需要较高的分辨率

方式：
- 划分普通的shadow map，只有在很重要的可见区域使用高分辨率
- 输入一系列的视点，并允许在这些视点上做阴影查询shadow query

特性：
- 视点驱动：层次格子结构基于用户的视点更新
- 程序可以限制内存使用
- 过程化：只要一个视点设置好了，那么图像的质量会一直提升，直到到达程序之前设置的内存

结构：
以tree的方式进行存储
- 每个节点包含一个固定的分辨率，并把shadow map划分成固定数量的cells
- 每个cell可能回包含其他的节点
  - 如果图片质量还可以提升，那么空的cell将被划分为新的节点
  - 也可以删除节点

## 2.1. 创建节点
用一个公式来作为划分cell的依据，根据公式算出来不太优先的就不划分节点。

公式实际上计算的是，当一个cell的shadow map 分辨率比视点分辨率要低，并且深度不连续的时候，计算这个cell中的可视点的数量。

<div align="center">

![][ASMCreateNodeCost]

</div>

## 2.2. 删除节点
使用LRU

[ASMCreateNodeCost]: ./ASMCreateNodeCost.jpg