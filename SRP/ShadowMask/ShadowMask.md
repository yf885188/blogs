<!-- TOC -->

- [1. 应用场景](#1-应用场景)
- [2. 烘焙阴影](#2-烘焙阴影)
    - [2.1. 流程](#21-流程)
    - [2.2. Occlusion Probes](#22-occlusion-probes)
- [3. 混合阴影](#3-混合阴影)
    - [3.1. distance shadowmask 模式](#31-distance-shadowmask-模式)
    - [3.2. shadowMask 模式](#32-shadowmask-模式)
- [4. 多光源](#4-多光源)

<!-- /TOC -->

# 1. 应用场景
- LightMap没有Shadow Max Distance的限制，但是不能渲染阴影
- 渲染的阴影不会被剔除掉，但是没有实现表现的效果
> ShadowMask使用场景:在Shadow Max Distance内用实时阴影，在Shadow Max Distance之外用ShadowMask，获得一个比较好的渲染体验。

# 2. 烘焙阴影
## 2.1. 流程
> Lighting Mode 选择ShadowMask -> ShadowMask Mode 选择 Distance ShadowMask -> 管线检测ShadowMask是否启用 -> Shadow中采样Shadow Mask 相关数据并加到GI中。

## 2.2. Occlusion Probes
动态物体要享受到ShadowMask的影响，只能通过Occlusion Probes来实现。

# 3. 混合阴影
## 3.1. distance shadowmask 模式
- 采样shadow mask
- 跟实时光的shadow进行混合
- 没有实时光情况下的处理

## 3.2. shadowMask 模式
跟distance shadowmask的区别
- 用ShadowMask的静态物体不再有realtime light的阴影（实际上是不采用混合的方式，两者取影响较小的那一个进行计算）

# 4. 多光源
- 最多支持4光源，用一个float4记录
