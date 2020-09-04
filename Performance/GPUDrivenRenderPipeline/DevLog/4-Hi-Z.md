可以参考以前的笔记[Hi-Z](../GPUDrivenRenderPipeline_References/Hi-Z/Note__Hi-Z.md)、[GPU Pro2中的实例](../GPUDrivenRenderPipeline_References/Practical,DynamicVisibilityForGames/Note_Practical,DynamicVisibilityForGames.md)、[Hi-Z的变体——层级遮挡视图](../GPUDrivenRenderPipeline_References/VirtualTexturing/Note__VirtualTexturing.md)等。

# 目标
- 生成Hi-Z maps
- 根据Hi-Z maps来剔除

# 思路
具体理论在之前的文首的笔记调研中都有介绍。本文采用的是最基础的Hi-Z实现，不用层级遮挡视图，也不用层次覆盖率视图做剔除。

work flow:
- 创建Hi-Z map array资源
- 生成最高精度的depth
- downsample 形成其他的Hi-Z map

# 实践
## mipmap生成
DX12不再有现成的GenerateMips函数，可以参考官方MiniEngine和[blog](https://slindev.com/d3d12-texture-mipmap-generation/)使用ComputeShader生成Mipmaps。

细节：
- 为了快速的获取4个采样点,设置Sampler中的D3D12_FILTER_REDUCTION_TYPE为D3D12_FILTER_REDUCTION_TYPE_MAXIMUM。
- 单层Mipmaps跟单个Texture的设置是一样的，只不过需要根据MipLevel设置MipSlice值

> 注意：
> - 遇到一个小坑，SV_Position在PS中的Z分量被设定为了1。需要单独存一份用于后续的深度计算。
> - 之前看论文有一种max/min reduction的采样方式，可以直接采样区域的Max值，但是要跟tile resource配合使用，这里down sample Hi-Z还是用loop的方式取4点进行比较。
> - 因为要绘制的是Occluder的深度，因此不先绘制sky box，清空buffer的时候用0，而不是类似DepthStencilView用1填充表示最远距离。