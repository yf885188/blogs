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
## 渲染方程
$$L_o(P, D_o) = L_e(P, D_o) + \int_{\Omega} f_r(P, D_i, D_o)L_i(P, D_i)(n \cdot D_i)dD_i$$

