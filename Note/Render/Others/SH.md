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
      \sqrt 2 K^m_l \sin(-m\phi)P^{-m}(\cos\theta) (m<0>)\\
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

展开为单参数形式：
$$f(\theta, \phi) = \sum^{\infty}_{l=0} \sum^l_{m=-l}C^m_lY^m_l(\theta, \phi) = \sum^{n^2}_{i=0} c_iy_i(s)$$

根据上面介绍的蒙特卡洛积分，将球谐系数$C_i$的积分形式转换为数值形式有：
$$C_i = \int_s light(v) Y_i(v){\rm d}s \\
\approx \frac{1}{N} \sum^N_{j=1}light(x_j)Y_i(x_j) \cdot 4\pi \\
= \frac{4\pi}{N} \sum^N_{j=1} light(x_j)Y_i(x_j)$$
其中，因为是球面的缘故，所以概率分布函数为$w(x_j) = \frac{1}{p(x_j)} = \frac{1}{\frac{1}{S}} = 4\pi$

之后，通过预计算的方式，计算出$C_i$，然后根据$f(s) =  \sum^{n^2}_{i=0} c_iy_i(s)$计算当前预计算光照。