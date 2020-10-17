
<!-- TOC -->

- [1. Deep Peeling](#1-deep-peeling)
  - [1.1. work flow](#11-work-flow)
  - [1.2. 优化](#12-优化)
    - [1.2.1. 双深度剥离](#121-双深度剥离)
    - [1.2.2. DS](#122-ds)
- [固定深度法](#固定深度法)

<!-- /TOC -->

# 1. Deep Peeling
[参考](https://zhuanlan.zhihu.com/p/92337395)

## 1.1. work flow

<div align="center">

![][DeepPeeling]

</div>

[DeepPeeling]: ./DeepPeeling.jpg

> - 多pass，分层进行深度剥离

## 1.2. 优化
缺点：
- 多pass绘制消耗太大

### 1.2.1. 双深度剥离
- 两个depth buffer，两个color buffer

### 1.2.2. DS
- 创建多个G-Buffer

# 固定深度法
解决一些特定的场景，比如，透明物体包裹