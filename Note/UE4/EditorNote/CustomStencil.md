# 简介
不同于系统使用的StencilDetphBuffer，类似于DepthOnlyPass新建了一个单独的管线和RT来做。

# UE4中的实现
资源：
- SceneContext.CustomStencilSRV:
  - 通过SnapshotSource.CustomStencilSRV初始化
  - RHICreateShaderResourceView 创建
  - Mobile上手动创建
- FSystemTextures.StencilDummySRV : 系统默认创建的Custom Stencil。
  - 使用RHICreateShaderResourceView创建，多线程创建，然后交给RHI。
  - 使用StencilDummy获取，StencilDummy的底层实现是一个Pool管理的RT的封装。记录的是创建SRV的一些基本信息，包括tex的基本创建信息，以及对应的allocator。
- FSystemTextures.InitializeCommonTextures ：所有texture资源创建的地方

> 如果启用了CustomDepth就会使用SceneContext中的CustomDepth和CustomStencilSRV
> 反之，用系统开始就初始化好的GSystemTextures.StencilDummySRV


资源设置接口：
- SetupSceneTextureUniformParameters 

封装的参数：
- SceneTextureParameters

# Render的过程
![][RenderFlow]

[RenderFlow]: ./RenderFlow.jpg

# 坑：
- 考虑创建的几个来源：
  - SceneContext: 如果场景开启了自定义深度的话，如果场景没有的话，默认使用GSystemTextures.StencilDummySRV。
    - SnapshotSource：并行的时候，进行快速数据拷贝的方式。最开始的CustomStencilSRV还是通过if (!(bHasValidCustomDepth && bHasValidCustomStencil))使用下面的两种方式创建的。
    - RHICreateShaderResourceView ： SnapshotSource无效，且CurrentFeatureLevel > ERHIFeatureLevel::ES3_1
    - 手动创建View:  SnapshotSource无效，且CurrentFeatureLevel <= ERHIFeatureLevel::ES3_1
  - FSystemTextures.StencilDummySRV: 没有开启自定义深度，开始初始化的时候。

- 移动设备上根据图形API的支持，需要选择是否要手动创建两个Buffer，而非DepthStencilBuffer

# 存在的问题
- 不管有没有启用Custom Stencil，都创建了2个stencil buffer：默认的和专门用于custom stencil。（没有考虑Mobile上的处理）