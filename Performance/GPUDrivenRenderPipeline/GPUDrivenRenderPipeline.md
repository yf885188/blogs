<!-- TOC -->

- [1. 准备](#1-准备)
- [2. 目标](#2-目标)
- [开发日志](#开发日志)

<!-- /TOC -->

# 1. 准备
[引用](./GPUDrivenRenderPipeline_References/References.md)

# 2. 目标
按照[Ubisoft这篇论文](./GPUDrivenRenderPipeline_References/References/aaltonenhaar_siggraph2015_combined_final_footer_220dpi.pdf)的思路搭一个管线。

主要目标——必须要有的技术：
- Deferred Texturing/Deferred Lighting
- Vertex Pulling Render Pipeline
- Hi-Z Occlusion Culling
- Mesh Cluster Rendering

次要目标——有时间就搞的技术：
- 多线程/多引擎框架
- Virtual Texture
- Shadow Caster Culling
  
# 开发日志
- [延迟渲染](./DevLog/1-DeferredTexturing.md)
- [场景树剔除](./DevLog/2-SceneTreeCulling.md)
- [RingBuffer替代帧资源](./DevLog/3-RingBuffer.md)
- [Hi-Z](./DevLog/4-Hi-Z.md)
- [ClusterRendering](./DevLog/5-MeshClusterRendering.md)
