[参考](https://zhuanlan.zhihu.com/p/66518450)

# 原理
IBL： 相当于用一个无限大的球面光源在照射场景。辐射度由 environment map 决定。

渲染方程中，就是对半球形的入射光线的光照效果进行积分，在非IBL的理论中，光源的数量都是有限的。IBL的理论把光线数量扩展到无穷，但是通过environment map这种离散的方式进行近似的模拟。

为了在实时渲染中加速积分运算，对积分部分进行了预渲染。主要分为：
- 漫反射部分： irradiance map —— 只跟法线方向有关
- 镜面反射部分： 
  - pre-filter environment map —— 跟粗糙度和光线反射方向有关
  - BRDF LUT —— 拆分为scale和bias，都是跟粗糙度和出射光线与法线的夹角有关

> - 镜面反射部分有2个变量，因此通常使用这两个变量的三线性采样来计算得到最终的采样值。

# Monte Carlo 积分
在很多光照计算中会涉及到积分运算，Monte Carlo是一种比较常见的处理方式。[参考](https://zhuanlan.zhihu.com/p/61611088)