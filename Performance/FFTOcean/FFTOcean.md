<!-- TOC -->

- [理论基础](#理论基础)
    - [频谱和FFT](#频谱和fft)
    - [海水频谱的统计学模型](#海水频谱的统计学模型)
    - [数学建模](#数学建模)
        - [时间的影响](#时间的影响)
        - [IDFT](#idft)
    - [法线](#法线)
    - [尖浪](#尖浪)
        - [挤压操作](#挤压操作)
        - [雅可比行列式](#雅可比行列式)
    - [IDFT的计算](#idft的计算)
        - [用IFFT计算IDFT](#用ifft计算idft)
        - [蝶式网络](#蝶式网络)
        - [bitreverse算法](#bitreverse算法)
    - [工程实现](#工程实现)

<!-- /TOC -->

实现一个FFT Ocean，主要参照[这个系列的blog](https://zhuanlan.zhihu.com/p/64414956),国内翻译得比较详尽的一套FFT Ocean教程，具体的推导过程这里不再赘述。

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

$F_0^*$ 表示$F_0$的共轭复数;

k表示$\vec k$的模;

$w(k) = \sqrt{gk}$,g为重力加速度。

$F_0(\vec k) = \frac {1}{\sqrt 2} (\xi_r + i\xi_i)\sqrt{P_h(\vec k)}$，$\xi_r$ 和$\xi_i$为相互独立的随机数，且均服从均值为0，标准差为1的正态分布，具体的算法在[博主的另一篇文章](https://zhuanlan.zhihu.com/p/67776340);

$P_h(\vec k) = A \frac {e^{\frac {-1}{{(kS)}^2}}}{k^4}|\vec k \cdot \vec {wind}|$，$\vec {wind}$表示风向，$S = \frac {V^2}{g}$，V表示风速

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
 $\vec N$ = normalize((0, 1, 0) - $(\nabla h_x(\vec x, t), 0, \nabla h_z(\vec x, t))$

### 尖浪
类似GensterWave进行挤压，根据雅可比行列式判断有向面积是否为负值来判断当前是否为尖浪的白沫。具体的推导看篇首参考的blog。
#### 挤压操作
$$\vec x_{after} = \vec x_{before} +\lambda \vec D(\vec x, t) $$
其中，
$$\vec D(\vec x, t) = \sum_{\vec k} -i\frac{\vec k}{k}h(\vec k,t)e^{i\vec k \cdot \vec x}$$
#### 雅可比行列式
$$J(\vec x) =\left| \begin{matrix} J_{xx} & J_{xz} \\ J_{zx} & J_{zz} \end{matrix} \right| $$
其中，
$$J_xx = \frac{\delta x^{\prime}}{\delta x} = 1 + \lambda\frac{\delta D_x(\vec x, t)}{\delta x}$$ 
 
 $$J_zz = \frac{\delta z^{\prime}}{\delta z} = 1 + \lambda\frac{\delta D_z(\vec x, t)}{\delta z}$$ 

 $$J_xz = \frac{\delta x^{\prime}}{\delta z} = \lambda\frac{\delta D_x(\vec x, t)}{\delta z}$$ 

 $$J_zx = \frac{\delta z^{\prime}}{\delta x} = \lambda\frac{\delta D_z(\vec x, t)}{\delta x}$$ 

如果$J(\vec x)$<0则表示是尖浪。
### IDFT的计算
这一节的内容主要是公式的推导，具体过程还是看篇首提到的blog，这里只写结论和相关知识点。
#### 用IFFT计算IDFT
基本思路：把N个输出拆分成$\frac N 2$的偶数输入和$\frac N 2$的奇数输入，分别进行计算之后进行汇总。
标准IDFT公式：
$$x(n)=\frac {1}{N}\sum_{k=0}^{N-1} X(k)e^{i\frac {2\pi kn}{N}}$$
通过IFFT计算的公式：
$$x(n) = \begin{cases} G(n) + W_N^{-n} H(n) \\ G(n - \frac {N}{2}) + W_N^{-n}H(n - \frac N 2) \end{cases}$$
其中，
$$G(n) = \frac 1 N \sum_{k=0}^{\frac N 2 - 1} g(k)e^{i\frac {2\pi kn}{\frac N 2}} = \frac 1 N \sum_{k=0}^{\frac N 2 - 1} x(2k)e^{i\frac {2\pi kn}{\frac N 2}}, n \in \{0, 1, ..., \frac N 2 - 1\}$$
$$H(n) = \frac 1 N \sum_{k=0}^{\frac N 2 - 1} h(k)e^{i\frac {2\pi kn}{\frac N 2}} = \frac 1 N \sum_{k=0}^{\frac N 2 - 1} x(2k+1)e^{i\frac {2\pi kn}{\frac N 2}}, n \in \{0, 1, ..., \frac N 2 - 1\}$$
$$W_n^k = e^{-i\frac{2\pi k}{N}}$$

#### 蝶式网络
上面的表现是一种迭代的方式，每次计算后的输出作为后面的输入再次进入后面的计算。另一种直观的表现方式就是蝶式网络，虽然计算上的过程差不多，但是会影响设计和理解。具体就是只有一次输入输出，把每次的计算展开。

<div align=center>

![IFFT][IFFTGraph]

IFFT 网络

![BufferflyIFFT][BufferFlyIFFT]

蝶式网络
</div>

#### bitreverse算法
需要确定x(n) 和 X(k)中n和k的对应关系，采用的这个算法。
对于在n位的x(n),经过N个点的蝶式网络之后，只需要把n化为$log_2 N$位的二进制数，然后reverse转为十进制，就得到了X(k)的k。

#### 联系
主要是将第一部分的相关公式给化成第二部分IFFT的相关形式。具体不在赘述。
$$A(u^{\prime}, v^{\prime}, t) = \frac {h((u^{\prime} - \frac M 2)* \frac L M, (v^{\prime} - \frac M 2), t)} {(-1)^{(v^{\prime} - \frac M 2)*\frac L M}} $$
$$B(u^\prime, v^\prime, t) = h^{\prime\prime}((u^\prime - \frac M 2) * \frac L M, m^\prime, t)(-1)^{m^\prime}$$
$$C(u^\prime, m^\prime, t) = \frac {h^{\prime\prime}((u^\prime - \frac M 2) * \frac L M, m^\prime, t)} {(-1)^{(u^\prime - \frac N 2) * \frac L M}}$$
$$D(n^\prime, m^\prime, t) = h^\prime(n^\prime, m^\prime, t)(-1)^{(n^\prime)}$$

其中，
$$h^\prime(n^\prime, m^\prime, t) = h(\frac {2\pi(n^\prime - \frac N 2)} L, \frac {2\pi(m^\prime - \frac N 2)} L, t)$$
$$h^{\prime\prime}(u^\prime - \frac N 2, m^\prime, t) = (-1)^{u^\prime - \frac N 2} \sum_{n^\prime = 0}^{N - 1}h^\prime(n^\prime, m^\prime, t)e^{i\frac {2\pi n^\prime (u^\prime - \frac M 2) \frac L M} N }$$

因此有：
$$A(u^\prime, v^\prime, t) = \sum_{m^\prime = 0}^{N-1}B(u^\prime, m^\prime, t)e^{i\frac {2\pi m^\prime (v^\prime - M / 2) * L/M} N} $$

$$C(u^\prime, m^\prime, t) = \sum_{n^\prime = 0}^{N - 1}D(n^\prime, m^\prime, t)e^{i\frac {2\pi n^\prime{v^\prime - N/2}*L/M } N}$$
在D中带入第一部分的统计学海洋频谱公式之后，由下而上使用IFFT即可计算得到$A(u^\prime, v^\prime, t)$即为最后的高度。

#### 工程实现
##### IFFT的实现
使用ComputeShader实现，把权重系数也即$W_m^n$提前计算做蝶形lut。
每一个阶段的输出会变成下一个阶段的输入，当前阶段的输入不会空置，之后可以转换成下一个阶段的输出，采用ping pong texture的方式，应对这种频繁输出的情况。
每个阶段的计算数据又是相互独立的，因此采用compute shader的方式在每个阶段进行并行计算，这样进一步减少了运算时间。

#### 蝶形lut的实现
因为是IFFT，所以这里的lut中存的其实是$W_N^{-k}$。而$W_N^{-k}$在采样方位确定之后是确定的，可以离线的方式先计算出来。
又有$W_N^{-k - N/2} = -W_N^{-k}$，这里的－放到ComputeShader中一起进行计算，所以在这里实际上只计算一次，存两次。
在开始确定了采样数之后，阶段数也能直接得出，这样的话，每个$W_N^{-k}$的计算实际上是相互独立的，在compute shader中能一次性都算出来。


[GradVecGraph]: ./grad_vec.jpg
[IFFTGraph]: ./ifft.jpg
[BufferFlyIFFT]: ./butterfly_ifft.jpg
