实现一个FFT Ocean，主要参照(这个系列的blog)[https://zhuanlan.zhihu.com/p/64414956],国内翻译得比较详尽的一套FFT Ocean教程，具体的推导过程这里不再赘述。

## 理论基础
### 频谱和FFT
一个海面可以理解成趋近于无限的三维位置点集合，每个点可以看作许多不同频率的正弦波进行的叠加。
点的位置可以看作是空域信号，记为f(x);
由频率计算的振幅和相位可以成为该点位置信号的频谱,记为F(w)。

>这里虽然说的是频率决定的是振幅和相位两个量，但是F(w)还是一维的，因为正弦的表达形式能直接将相位位移拆分成振幅的影响，这个在推导过程中可以看到。

由f(x)求F(w),称为傅里叶变换FFT；
反之为傅里叶逆变换IFFT。

具体的推导过程看本文开头给的链接，最后得到的两者的关系为

$$f(x) = \sum_{w} F(w)e^{iwx}$$

### 海水频谱的统计学模型
根据海洋统计学得到的海水频谱模型

$$F(\vec k,t) = F_0(\vec k)e^{iw(k)t} + F_0^*(-\vec k)e^{-iw(k)t} $$

其中,

$\vec k$表示频谱中的采样坐标;

$F_0^*$ 表示F_0的共轭复数;

k表示$\vec k$的模;

$w(k) = \sqrt{gk}$,g为重力加速度。

$F_0(\vec k) = \frac {1}{\sqrt 2} (\xi_r + i\xi_i)\sqrt{P_h(\vec k)}$，$\xi_r$ 和$\xi_i$为相互独立的随机数，且均服从均值为0，标准差为1的正态分布，具体的算法在[博主的另一篇文章](https://zhuanlan.zhihu.com/p/67776340);

$P_h(\vec k) = A \frac {e^{\frac {-1}{{kS}^2}}}{k^4}|\vec k \cdot \vec {wind}|$，$\vec wind$表示风向，$S = \frac {V^2}{g}$，V表示风速

### 数学建模
#### 时间的影响
因为海洋频谱的统计学模型中加入了时间t，并且海面的表现处于一个三维场景中，需要计算的实际上是一个高度分布，所以后续的式子可以表现为如下：
$$h(\vec x,t) = \sum_{w} F(\vec k, t)e^{iwx}$$
 
#### IDFT
因为计算机无法处理无限连续的点进行处理，所以这里采用的是离散采样的方式，也即离散傅里叶逆变换IDFT。

采样点的选择（原文空间位置和频域采样点数一致，私以为不用）：

频谱： 保证在一个patch中能覆盖一个$2\pi$的频率范围，这样之后的patch拼接会相对方便一些。采样(M,M)尺寸范围上的点，可以得到频谱的采样点公式为

$$\vec k = ({\frac {x \cdot 2\pi}{L}}, \frac {y \cdot 2\pi}{L})$$

其中$x,y \in\{ -\frac{M}{2}, ...., \frac{M}{2}-1\}$。

空间位置：跟频谱类似，采样在(N,N)尺寸范围上的点。采样公式为：
$$\vec x = (\frac{x\cdot L}{N}, \frac{z\cdot L}{N}) $$
其中$x,y \in\{ -\frac{N}{2}, ...., \frac{N}{2}-1\}$。

### 法线
根据梯度向量和Up向量确定，均与频谱无关，频谱相关的处理按照常数。

<div align=center>

![法线、梯度向量和Up向量的关系][GradVecGraph]

</div>

根据高度函数可直接求得梯度向量：

 $$ \nabla h(\vec x, t) = \sum_{\vec k} i\vec k h(\vec k, t)e^{i\vec k\cdot\vec x}$$
 
 法线向量为：
 $\vec N$ = normalize((0, 1, 0) - $(\nabla h_x(\vec x, t), 0, \nabla h_z(\vec x, t))$)

 ### 尖浪
 类似GensterWave进行挤压，根据雅可比行列式判断有向面积是否为负值来判断当前是否为尖浪的白沫。具体的推导看篇首参考的blog。
 #### 挤压操作
 $$\vec x_{after} = \vec x_{before} +\lambda \vec D(\vec x, t) $$
 其中，
$$\vec D(\vec x, t) = \sum_{\vec k} -i\frac{\vec k}{k}h(\vec k,t)e^{i\vec k \cdot \vec x}$$
#### 雅可比行列式
$$J(\vec x) =\left| \begin{matrix} J_xx & J_xz \\ J_zx & J_zz \end{matrix} \right| $$
其中，
$$J_xx = \frac{\delta x^{\prime}}{\delta x} = 1 + \lambda\frac{\delta D_x(\vec x, t)}{\delta x}$$ 
 
 $$J_zz = \frac{\delta z^{\prime}}{\delta z} = 1 + \lambda\frac{\delta D_z(\vec x, t)}{\delta z}$$ 

 $$J_xz = \frac{\delta x^{\prime}}{\delta z} = \lambda\frac{\delta D_x(\vec x, t)}{\delta z}$$ 

 $$J_zx = \frac{\delta z^{\prime}}{\delta x} = \lambda\frac{\delta D_z(\vec x, t)}{\delta x}$$ 

[GradVecGraph]: ./grad_vec.jpg
