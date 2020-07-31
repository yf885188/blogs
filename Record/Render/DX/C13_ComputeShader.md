可参考OpenCL。

# GPGPU

# 线程与线程组
- 共享内存，组内可用，组间不可用

# 计算着色器的构成
- 通过常量缓冲区访问的全局变量
- 输入与输出资源
- [numbthreads(x,y,z)]属性来指定线程组
- 线程组要执行的着色器指令
- 线程ID系统值参数

> 使用计算流水线状态对象

# work flow
## 输入与输出
- 纹理输入：绑定到SRV，并绑定到RootSignature
  - 只能只读

## 纹理输出与UAV
- 纹理输出与UAV绑定
  - 可读可写

## 纹理采样的两种方式
- 整数索引：数组形式的采样
- SampleLevel: 纹理UV形式的采样
  - 归一化的UV坐标
  - mipmap层级的过滤处理

## 结构化缓冲资源
- StructuredBuffer

## 将ComputeShader的计算结果拷回内存
- 设置堆属性为D3D12_HEAP_TYPE_READBACK,然后通过ID3D12GraphicsCommandList::CopyResource复制到系统内存资源中
- 保证系统内存资源与带复制的资源有相同的类型和大小
- 用映射API将内存资源的数据映射成类型的内存块。

# 线程标识的系统值
- 线程组ID： SV_GroupID
- 组内线程ID: SV_GroupThreadID
- 调度线程ID: SV_DispatchThreadID

# 追加缓冲区与消费缓冲区
不用管数据在缓冲区中的执行顺序，与最后的输出索引顺序。
- 消费缓冲区： ConsumeStructuredBuffer，输入缓冲区
- 追加结构化缓冲区： AppendStructuredBuffer, 输出缓冲区

# 共享内存与线程同步
- 共享内存：线程组内共享的内存。所有的共享内存总和上限为32KB。
- 线程同步：GroupMemoryBarrierWithGroupSync()

# 细节记录
- 高斯模糊的分离性，让n*n的采样数量变为n+n的采样数量，降低了纹理采样带来的性能消耗。