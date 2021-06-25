# 数据并行架构
CPU可用的避免阻塞的好方式：分支预测，指令记录，寄存器重命名，缓存预取。而GPU则是使用一种并行的方式。

## SIMD
纹理采样：所有的fragment在遇到纹理采样的时候，会停下来，然后切换fragment处理，等所有的fragment处理完之后，会返回到第一个fragment纹理采样的地方，用这种并行的方式，来加速纹理采样的速度。

这种并行方式的进一步延伸就是SIMD。

### 架构
- thread : memory for input，register space
- warps/wavefronts : threads bundled into groups

> 和之前的理解不一样的地方是，这里的SIMD并不是指一组group的processor一直为一个warp/wavefronts服务。为了降低stall，通常会在一组warp/wavefronts等待memory fetch的时候，会swap另外一组warp/wavefronts进来进行计算。
> 并且，因为thread自带memory，因此这类swap很快。
> 但是如果warps/wavefront的数量太少，swap不够，延迟的问题就又会暴露出来。

### shader结构如何影响效率
- 每个thread使用register的数量。GPU register的数量恒定，一个shader的register越多，warps/wavefront越少，swap越少。
- 内存访问的频率。
- 动态分支。thread divergence问题：同一个warps/wavefronts中的一个thread忙，其他的忙等。

![][SIMD]

[SIMD]: ./images/SIMD.png

# VS
- uniform分配的寄存器比inputs或者outputs分配的寄存器数量要多很多
  - 主要是因为inputs和outputs要为每个vs分开存储

![][VSRegisters]

[VSRegisters]: ./images/VSRegisters.png


- 流程控制：高级语言层次。if 和 case等等
  - 静态流程控制：依据uniform inputs。意味着draw call间的流程控制都是一样的。
  - 动态流程控制：基于变化的输入（varying inputs），也更耗性能。

> instancing : 使用不同的object data绘制一个object/mesh多次，并且只用一个draw call。
# 
用处：
- 生成/变形object
- 蒙皮
- 过程化变形
- 创建粒子
- 一些物理现象等等
- 地形

# ts
通常都包含3个方面：
- hull shader
- tessellator
- domain shader

![][TSShadersWorkFlow]

[TSShadersWorkFlow]: ./images/TSShadersWorkFlow.png

# gs
- 更改图元类型。
- 改变顶点数，但是不会生成多的顶点。
- 改变输出数据或者对复制做限制。
- 使用instancing

> - 可以保证输出的顺序
> - 只有三个地方能产生GPU work：光栅化、ts和gs。而gs因为是完全可编程的存在，所以具有最少的可预测性。而且gs也很少能体现GPU的实力，在一些移动设备上也通常用软件的方式实现gs，在这些场合下，也不鼓励gs的使用。

# streaming out
Shader Model 4.0的新特性。vs/ts/gs出来的数据可以通过关掉的光栅化的方式，以stream的方式传递、迭代或者read back。在某些特定领域：flowing water、粒子特效等方面有应用。

缺陷：
- 只能以float的方式返回，有性能消耗
- 作用于图元，而非顶点
- 不支持顶点共享，因此通常都是把三角形按照点集图元进行传递

# ps
