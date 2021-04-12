[参考](https://zhuanlan.zhihu.com/p/49436452)

<!-- TOC -->

- [1. 原因](#1-原因)
- [2. 渲染方程](#2-渲染方程)
- [3. 蒙特卡洛积分](#3-蒙特卡洛积分)
  - [3.1. 思路](#31-思路)
  - [3.2. 平均分布的概率函数的确定](#32-平均分布的概率函数的确定)
- [4. SH](#4-sh)
  - [4.1. 谐波](#41-谐波)
  - [4.2. 伴随勒让德多项式（ALP）](#42-伴随勒让德多项式alp)
    - [4.2.1. 勒让德多项式](#421-勒让德多项式)
    - [4.2.2. 伴随勒让德多项式（ALP）](#422-伴随勒让德多项式alp)
  - [4.3. SH](#43-sh)
  - [4.4. 球面函数使用SH表示](#44-球面函数使用sh表示)
  - [4.5. 特性](#45-特性)
    - [4.5.1. 标准正交性](#451-标准正交性)
    - [4.5.2. 旋转不变性](#452-旋转不变性)
    - [4.5.3. 函数乘积的积分=球谐系数向量的点积](#453-函数乘积的积分球谐系数向量的点积)
- [5. 基于SH的PRT](#5-基于sh的prt)

<!-- /TOC -->

# 1. 原因
- 球面上算渲染方程的积分值
- 使用蒙特卡洛积分把积分运算变成离散的数值运算

# 2. 渲染方程

# 3. 蒙特卡洛积分
## 3.1. 思路
$f(x)$的数学期望：
$$E[f(x)]=\int f(x)p(x){\rm d}x$$
可以转换为：
$$I=\int \frac{f(x)}{p(x)}p(x){\rm d}x=E(\frac{f(x)}{p(x)})$$
分析方式可以转换为数值方式：
$$I\approx \frac{1}{N}\sum ^N_{i=1} \frac{f(x_i)}{p(x_i)}$$

需要确定：
- 球面上是通过两个参数$\theta$和$\phi$来确定
- 概率函数需要的是平均分布

## 3.2. 平均分布的概率函数的确定
- 根据期望反推:
  - $$\iint _S{\rm d}A=4\pi$$
  - 所以，希望概率函数是：
  - $$p(x) = \frac{1}{4\pi}$$
- 又因为
  - $${\rm d}A = \sin\theta{\rm d}\theta{\rm d}\phi$$
- 所以概率函数
  - $$p(\theta,\phi)=\frac{1}{4\pi}\sin\theta$$
- 对应的分布函数为：
  - $$F_{\theta}(\theta) = \int ^{\theta}_0 f(\hat {\theta}){\rm d}\hat{\theta} = \frac{1-\cos\theta}{2}$$
  - $$F_{\phi}(\phi) = \int ^{\phi}_0 f(\hat {\phi}){\rm d}\hat{\phi}=\frac{\phi}{2\pi}$$
- 使用Inverse Transform Sampling算法，把一个均匀分布变量$U\backsim Uniform[0,1]$，再给定一个我们想要生成的连续随机变量$X$及其理想分布函数$F_X(x)$,我们有
  - $X = F^{-1}_X(U)$，则$X\backsim F_X(x)$
- 综上，存在这样的映射：
  - $$
  \left\{
      \begin{array}{c}
      \theta = \arccos(1-2\Epsilon _x) \\
      \phi = 2\pi \Epsilon _y \\
      \end{array}
    \right.
  $$

# 4. SH
基于伴随勒让德多项式的谐波基函数表示。

## 4.1. 谐波
- 不同的频率函数拟合模拟原始信号
- 一般用互相正交的函数作为基函数来拟合
  - > 例如，FFT的基函数是通过欧拉方程展开的$\sin \theta$和$\cos \theta$

## 4.2. 伴随勒让德多项式（ALP）
### 4.2.1. 勒让德多项式
- 勒让德微分方程的解
  - 物理场景中一类常微分方程
  - 拉普拉斯微分方程的一种变形
  - 球坐标中求解拉普拉斯微分方程时，问题常簋街为勒让德方程

形式：
$$P_n(x) = \frac{1}{2^n  n!} \frac{d^n}{{\rm d}x^n}[(x^2-1)^n]$$

### 4.2.2. 伴随勒让德多项式（ALP）
根据勒让德多项式得到：
$$P^m_l(x)=(-1)^m(1-x^2)^{\frac{m}{2}}\frac{d^m}{{\rm d}x^m}(P_l(x))$$

其中：
- $l$表示ALP的“degree/band index”,m表示“order”;
- $m \in [0, l]$。

> 剩下的推导等相关内容，参看原文。

## 4.3. SH
基于ALP的谐波基函数表示。
表示：
$$Y^m_l(\theta, \phi) = \left\{
      \begin{array}{c}
      \sqrt 2 K^m_l \cos(m\phi)P^m_l(\cos\theta) (m>0)\\
      \sqrt 2 K^m_l \sin(-m\phi)P^{-m}(\cos\theta) (m<0)\\
      K^0_lP^0_l(\cos\theta) (m=0)\\
      \end{array}
    \right.
    $$

其中：
- $P^m_l表示的是ALP$
- $K^m_l = \sqrt {\frac{2l+1}{4\pi} \frac{(l-|m|)!}{(l+|m|)!}}$
- 跟ALP不同，$m \in [-l, l]$

![][SHVisualize]

[SHVisualize]: ./images/SHVisualize.jpg

使用单参数的方式把SH函数的项给展开。
$$y_i(\theta, \phi) = Y^m_l(\theta, \phi)$$
其中，$i=l(l+1)+m$

## 4.4. 球面函数使用SH表示
$f(\theta,\phi)$在球面$S$上的展开式为：
$$f(\theta, \phi) = \sum^{\infty}_{l=0} \sum^l_{m=-l}C^m_lY^m_l(\theta, \phi)$$
其中，$C^m_l = \int_s f(s) Y^m_l(s){\rm d}s$

> 求$C^m_l$的方式,有的blog上是通过对上面的积分，两边乘以$Y^m_l$然后积分的方式得到。这种方式有误，应该从物理意义上来理解，通常叫投影。如下图
> ![][SHCoffients]

[SHCoffients]: ./images/SHCoffients.jpg

> 那是不是可以这么理解：$C^m_l$实际上是把函数$f(s)$在球面上的所有的值进行编码之后分散到频带上的系数。

展开为单参数形式：
$$f(\theta, \phi) = \sum^{\infty}_{l=0} \sum^l_{m=-l}C^m_lY^m_l(\theta, \phi) = \sum^{n^2}_{i=0} c_iy_i(s)$$

根据上面介绍的蒙特卡洛积分，将球谐系数$C_i$的积分形式转换为数值形式有：
$$C_i = \int_s light(v) Y_i(v){\rm d}s \\
\approx \frac{1}{N} \sum^N_{j=1}light(x_j)Y_i(x_j) \cdot 4\pi \\
= \frac{4\pi}{N} \sum^N_{j=1} light(x_j)Y_i(x_j)$$
其中，因为是球面的缘故，所以概率分布函数为$w(x_j) = \frac{1}{p(x_j)} = \frac{1}{\frac{1}{S}} = 4\pi$

之后，通过预计算的方式，计算出$C_i$，然后根据$f(s) =  \sum^{n^2}_{i=0} c_iy_i(s)$计算当前预计算光照。

## 4.5. 特性
- 标准正交性
- 旋转不变性
- 函数乘积的积分=球谐系数向量的点积

> 具体的旋转推导可以看原文

### 4.5.1. 标准正交性
$$
  \left\{
    \begin{array}{c}
    \int y_iy_j = 1 (i=j) \\
    \int y_iy_j = 0 (i\neq j) \\
    \end{array}
  \right.
$$

### 4.5.2. 旋转不变性
$$R(f(s))=f(R(s))$$
也即，直接转换函数本身或者把自变量旋转之后再通过函数计算，得到的结果是一样的。保证了在旋转物体灯光的时候，光强不会产生不良变化。

### 4.5.3. 函数乘积的积分=球谐系数向量的点积
基于上面提到的正交性等特性配合投影操作，有如下的变换
$$L(s)t(s)=(\sum_{i=0}^{n^2}c_{li}y_i(s))(\sum_{i=0}^{n^2}c_{ti}y_i(s))\\=\sum_{i=0}^{n^2}(c_{li}y_i(s))(c_{ti}y_i(s))\\=\sum_{i=0}^{n^2}L_i(s)t_i(s)$$
其中,$c_{li},c_{ti}$分别对应$L(s),t(s)$的SH cofficients。

因此有，
$$\int_S L(s)t(s){\rm d}s = \sum_{j=0}^{sampler\_count}\sum_{i=0}^{n^2}L_it_i$$
（原链接有误，少了一个积分）


# 5. 基于SH的PRT
分为两个部分**传输**和**着色**

以不考虑阴影的漫反射传输为例
光照方程为：
$$L_{DU} = \frac{\rho_x}{\pi} \int_S L_i(x, \omega_i)max(N_x\cdot \omega_i, 0){\rm d}\omega_i$$

基于**函数乘积的积分=球谐系数向量的点积**，就有
$$L_{DU} =\frac{\rho_x}{\pi} \sum_{j=0}^{sampler\_count}\sum_{i=0}^{n^2} L_i*max(N_x\cdot w_i,0) $$

