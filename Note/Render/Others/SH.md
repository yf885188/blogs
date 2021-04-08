[参考](https://zhuanlan.zhihu.com/p/49436452)

# 原因
- 球面上算渲染方程的积分值
- 使用蒙特卡洛积分把积分运算变成离散的数值运算

# 渲染方程

# 蒙特卡洛积分
## 思路
$f(x)$的数学期望：
$$E[f(x)]=\int f(x)p(x){\rm d}x$$
可以转换为：
$$I=\int \frac{f(x)}{p(x)}p(x){\rm d}x=E(\frac{f(x)}{p(x)})$$
分析方式可以转换为数值方式：
$$I\approx \frac{1}{N}\sum ^N_{i=1} \frac{f(x_i)}{p(x_i)}$$

需要确定：
- 球面上是通过两个参数$\theta$和$\phi$来确定
- 概率函数需要的是平均分布

## 平均分布的概率函数的确定
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

