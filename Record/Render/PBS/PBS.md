<!-- TOC -->

- [1. 物理规律](#1-物理规律)
- [2. BRDF 双向反射分布函数](#2-brdf-双向反射分布函数)
  - [2.1. 辐射亮度和辐射入射度](#21-辐射亮度和辐射入射度)
  - [2.2. BRDF 表示方程](#22-brdf-表示方程)
  - [2.3. 物理规律](#23-物理规律)
  - [2.4. 最终表示方程](#24-最终表示方程)
  - [2.5. Cook_Torrance](#25-cook_torrance)
- [3. Fresnel反射](#3-fresnel反射)
  - [3.1. Schlick 反射光强比值计算公式](#31-schlick-反射光强比值计算公式)
- [4. 微表面理论和Cook_Torrance模型](#4-微表面理论和cook_torrance模型)
  - [4.1. 微表面理论](#41-微表面理论)
  - [4.2. 法线分布函数](#42-法线分布函数)
  - [4.3. 几何衰减因子](#43-几何衰减因子)
  - [4.4. fresnel方程](#44-fresnel方程)
  - [4.5. Cook-Torrance BRDF 模型](#45-cook-torrance-brdf-模型)
- [5. Render Equation](#5-render-equation)

<!-- /TOC -->
# 1. 物理规律
- 光的折射和折射率
- 均匀介质
- 散射：介质不均匀，缓慢且连续变化，光的传播路径变成曲线；遇到介质中的杂质，光线就会被分成几束，发生散射。
- 能量守恒
  
# 2. BRDF 双向反射分布函数
描述的是光线照射到物体表面之后，散射和镜面反射后光线如何传播的问题。实际上是入射光线能量和从表面出射光线能量的比例关系。

用数学可以描述为：
当沿立体角$\vec{w_i}$的方向入射到点p的光辐射亮度为$L_i(p, \vec{w}_i)$时，计算朝向观察者的立体角$\vec{w_0}$方向上有多少辐射亮度$L_o(p, \vec{w}_o)$离开点p。

## 2.1. 辐射亮度和辐射入射度
- 辐射亮度$E(p, \vec{w_i})$：辐射表面在其单位投影面积的单位立体角内发出的辐射通量。
- 辐射入射度$L_i(p, \vec{w_i})$：单位面积被照射的辐射通量。

存在关系：
$$ dE(p, \vec{w_i}) = L_i(p, \vec{w_i}) * cos{\theta_i} d\vec{w_i} $$

## 2.2. BRDF 表示方程
$$f_r(p, \vec{w_o}, \vec{w_i}) = \frac {dL_o(p, \vec{w_o})} {L_i(p, \vec{w_i}) * cos{\theta_i} d\vec{w_i}}$$

## 2.3. 物理规律
- 交换律：交换出射、入射方向，BRDF 方程计算值不变。（只跟表面有关）
- 能量守恒

## 2.4. 最终表示方程
- 用出射入射方向代替原有的位置跟入射光方向
- 不同频率光的BRDF值不同，BRDF函数记录RGB三个分量的不同BRDF值
- 有限光源的情况

得到最终的表示方程：
$$L_o(\vec{v}) = \sum_{k=1}^n f(\vec{l_k}, \vec{v}) \oplus E_{l_k} cos{\theta_{i_k}}$$

## 2.5. Cook_Torrance 
使用辐射度学和微表面理论建立的BRDF模型。

# 3. Fresnel反射
近强折射+远强反射的效果。

## 3.1. Schlick 反射光强比值计算公式
一种近似的计算fresnel反射光强比值的公式。
$$R_{Schlick} = R_0 + (1 - R_0)\{1 - cos[max(\theta_i, \theta_t)]\}^5$$
- $\theta_i$ 为入射角
- $\theta_t$ 为折射角
- $R_0$ 为基础反射比例，$\theta_i$和$\theta_t$均为0时的反射比例。
  
> 导体和电解质的基础反射比例差别很大，因此材质也分为了金属工作流和非金属工作流，并添加了一个金属度属性。

# 4. 微表面理论和Cook_Torrance模型
## 4.1. 微表面理论
物体表面都是由一个个微表面构成，存在更多的物理规律应用场景。

## 4.2. 法线分布函数
当给定一个半角向量$\vec{h}$时，法线分布函数会返回其法线“恰好是”$\vec{h}$的微表面的个数占待光照计算表面区域中微表面总数的比例。

Unity 3D使用Trowbridge-Reitz GGX函数做法线分布函数：
$$D(\vec{h}) = \frac {roughness^2} {\pi * [(\vec{n} \cdot \vec{h})^2 (roughness^2 - 1) + 1]^2}$$

## 4.3. 几何衰减因子
微表面间的互相遮挡、反射等因素导致。

Smith-Schlick表达式：
$$\left\{
\begin{aligned}
K & = & \frac {roughness^2} {2} \\
G(\vec{w_i}, \vec{w_o}) & = & \frac {1} {(\vec{n} \cdot \vec{w_i}) \cdot (1 - K) + K} \frac {1} {(\vec{n} \cdot \vec{v}) \cdot (1 - K) + K}  \\
\end{aligned}
\right.$$

## 4.4. fresnel方程
$$F(\vec{h}, w_i) = C_{spec} + (1 - C_{spec})(1 - \vec{h} \cdot \vec{w_i})$$

## 4.5. Cook-Torrance BRDF 模型
镜面反射部分：
$$\left\{
\begin{aligned}
\vec{h} & = & \frac {\vec{w_o} + \vec{w_i}} {||\vec{w_o} + \vec{w_i}||} \\
f_{specular} & = & \frac {D(\vec{h})F(\vec h, \vec w_i )G(\vec{w_i},\vec{w_o})} {4(\vec n \cdot \vec{w_i})(\vec n \cdot \vec{w_o})}
\end{aligned}
\right.$$

漫反射部分：
$$f_{diffuse}(\vec{w_i}, \vec{w_o}) = \frac c \pi$$

Disney 漫反射部分：
$$\left\{
\begin{aligned}
f_{D90} & = & 0.5 + 2*roughness*(\vec h * \vec{w_i})^2    \\
f_{DisneyDiffuse}(\vec{w_i}, \vec{w_o}) & = & \frac c \pi  [1 + (f_{D90} - 1)(1 - \vec n \cdot \vec{w_i})^5] [1 + (f_{D90} - 1)(1 - \vec n \cdot \vec{w_o})^5]\\
\end{aligned}
\right.$$

整体的表达式：
$$f_{cook-torrance} = K_df_{diffuse} + K_sf_{specular}$$

# 5. Render Equation
[参考](https://www.zhihu.com/question/29182519/answer/43457554)

$$L_o(\vec x, \vec{w_o}, \lambda, t) = L_e(\vec{x}, \vec{w_o}, \lambda, t) + \int_{\Omega} f_r(\vec{x}, \vec{w_i}, \vec{w_o}, \lambda, t) L_i(\vec{x}, \vec{w_i}, \lambda, t) (\vec{w_i}, \vec n)$$