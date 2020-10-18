[参考](https://zhuanlan.zhihu.com/p/66518450)

# 原理
IBL： 相当于用一个无限大的球面光源在照射场景。辐射度由 environment map 决定。

渲染方程中，就是对半球形的入射光线的光照效果进行积分，在非IBL的理论中，光源的数量都是有限的。IBL的理论把光线数量扩展到无穷，但是通过environment map这种离散的方式进行近似的模拟。

为了在实时渲染中加速积分运算，对积分部分进行了预渲染。主要分为：
- 漫反射部分： irradiance map
- 镜面反射部分： 
  - pre-filter environment map
  - BRDF LUT

# Monte Carlo 积分
在很多光照计算中会涉及到积分运算，Monte Carlo是一种比较常见的处理方式。[参考](https://zhuanlan.zhihu.com/p/61611088)