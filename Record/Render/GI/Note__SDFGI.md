[参考](./SDFGI.pdf)

# 引言
已有的光照方案：
- Baked Light Map : 为静态光和静态场景准备
- Precomputed Radiance Transfer : 不受动态光源限制
- ...

但是，以上的方案都不能应对动态GI，并且会有预计算的的额外代价。

应对动态GI的解决方案：
- Voxel-Based GI
- Ray Tracing GI
- Screen Space GI
- RTXGI

但是这些都存在其他的问题。因此，提出了SDFDDGI这套框架。

主要思路：
- 采样空间域的辐照度函数
- 通过对采样结果进行插值来估计GI
- 为了使离散的采样结果和原本的辐照度函数尽可能的接近，使用SDF来控制采样点的分布
- 在插值采样结果的时候，我们也是用sdf来做可见性判断，来防止light leak现象的发生
- 也使用Contact GI来加强SDFDDGI中的细节
  
本文的目标：
- 使用SDF图元或者簇来压缩场景中的信息，减少数据量，加快计算
- SDF能非常快的访问最近的距离和空间梯度，来优化离散采样点的位置，来更好的适应辐射度方程的空间分布
- SDF也能比较高效的计算软阴影，优化离散点的采样策略，兵能完全规避掉light leaking现象，也会提供间接的软阴影效果

# 铺垫
渲染方程
$$L_o(P, D_o) = L_e(P, D_o) + \int_{\Omega} f_r(P, D_i, D_o)L_i(P, D_i)(n \cdot D_i)dD_i$$

- VirtualPointLights(VPL)
- Relfective Shadow Maps(RSM)
- Light Propagation Volumes(LPV)
- Ray Tracing GI
- Screen Space GI
- RTXGI:用光线追踪的配套硬件实现

# SDF
## 基本思路
- SDF可以用来简化低频的GI表现。
- SDF是空间中的一个标量域，代表空间内一点到场景中最近平面的距离。
- 如果点在最近平面的外面，则用正值表示。反之用负值表示。
- 根据上面三点来产生一个场景几何信息的紧凑展示。

## 优势
- 不要预计算
- 处理动态几何体和动态光线，也包括动画和skylight
- 帧间稳定性，对于动态改变的反应延迟很低
- 杜绝了light leaking问题
- 不需要特殊硬件，可运行于低段硬件

## work flow
- SDF Cluster : 生成场景的SDF表示和cluster 加速结构
- 选择Probe ：选择空间内合适的采样点
- Probe更新：更新采样点周围球形方向的辐照度
- 每像素的GI Shading：屏幕空间内，在不同的探针间插值来计算GI

## SDF Cluster
- 不存在volume textures中：平衡纹理分辨率，场景细节和数据量。
- 使用SDF基元组成的距离场，然后通过解析场景内全局的SDF来描述场景，而不再使用场景体素化。
> SDF基元：
> - 基元类型：矩形区域，平面，椎体或者其他方便计算SDF的可分析形式。
> - 相关的变换
- 每帧中，使用CPU来剔除相机周围的SDF，并未更远的基元使用LOD：保证使用最少的基元来解释一个更大的场景。
- 使用Cluster来加速SDF的查询

## Probe 选择
Irradiance函数的表示：
$$P = [x,y,z], D = [w, \theta] $$
$$Irradiance = E(P,D)$$

通过这样的方式把场景划分为多个Probe，每个Probe就存储到他位置附近的球面方向的辐射度。

> 空间域中的辐射度分布不是连续的，比如在墙面或者碰撞点就会存在很明显的不连续，这也是很多GI算法中light leaking的主要原因。因此需要谨慎的放置探针。

work flow:
<div align="center">

![][ProbeChoosingWorkFlow]

</div>

不需要人为的处理，也减少了动态物体带来的leaking。

[ProbeChoosingWorkFlow]: ./ProbeChoosingWorkFlow.png


## Probe Update
- 使用Compute Shader来加速update每个选中的probe。
- 通过对Cluster进行求交运算并做剔除，来加速Probe的更新。
- 使用2LastSDF加速求交判断（存疑？）
- 通过复用上一帧的GI数据，可以获取多Cell空间的GI
- 使用octahedral mapping来把辐射度数据存到探针图集纹理中

## 每像素的GI Shading
### Probe可见性测试（存疑……）
- SDF Shadow Trace（待研究）
- 对depth buffer进行downsample


## Contact GI
基于Ground Truth Ambient Occlusion（GTAO）的方式来加强diffuse GI细节。

在计算AO的同时，采样遮挡点的灯光，并把它跟探针GI进行合并，这样就能考虑到多次反射光线的影响，提高GI的LOD。

## Cascade Volume
用分布密度可变的probe volume，形成一个分层的级联Probe Volume，也即一个松散放置的probe volume，覆盖从相机到很远距离的区域。

- 节省的探针消耗可以用于增强GI效果。
- 使用平均值坐标（Mean Value Coordinates）来替代三线性插值来获得一个比较柔和的转变效果







