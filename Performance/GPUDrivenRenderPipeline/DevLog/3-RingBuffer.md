# 思路
## 资源的重新组织
参考[官方文档](https://docs.microsoft.com/zh-cn/windows/win32/direct3d12/memory-management)

## ring buffer 
原来的实现中，buffer的结构如下：
- VertexBuffer: 唯一。只在初始化的时候更新，其他时候不更新
- IndexBuffer: 同VertexBuffer
- FrameResource： 一般都是设置多个
  - MatBuffer: non constant buffer
  - ObjectBuffer: constant buffer
  - PassBuffer: constant buffer

现在把以上buffer都合并成一个buffer进行处理。并放到ring buffer中进行处理。

> 这里的处理在[后续](./5-MeshClusterRendering.md)中又有变化。

> 多帧的资源公用一个ring buffer，不像之前定死是N个帧资源，根据每帧的实体数量都会有不同的变化。但是ID3D12CommandAllocator还是要多配置几个用于缓冲。为了达到这种效果，又能保证ID3D12CommandAllocator可以重复使用，需要在帧资源比ID3D12CommandAllocator的资源多的时候对CPU进行阻塞。
> 也即是说，使用ring buffer进行阻塞的条件有两种：
> - ring buffer资源不够
> - cmd allocator的资源不够

# 表现
使用ring buffer替代原有的多UploadBuffer的FrameResource之后，帧率有20~30左右的提升。

