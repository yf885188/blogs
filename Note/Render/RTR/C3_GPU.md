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