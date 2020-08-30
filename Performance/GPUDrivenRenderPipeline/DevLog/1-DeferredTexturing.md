# logging
接入[spdlog](https://github.com/gabime/spdlog)

# 模型数据读入
使用fbx sdk导入fbx模型，[参考](http://help.autodesk.com/view/FBX/2020/ENU/)。

# 几种deferring rendering的比较
- [deferred shading](../GPUDrivenRenderPipeline_references/DeferredShading/Note__DeferredShading.md):两个pass
  - 第一个pass计算出16个float的属性：贴图等属性也要在这个阶段进行处理
  - 第二个pass根据这些属性来做pixel shader
- [deferred lighting](http://www.realtimerendering.com/blog/deferred-lighting-approaches/):三个pass。相比deferred shading对带宽要求更低
  - 准备G-Buffer(Depth和Normal)
  - 进行deferred lighting并得到L-Buffer
  - 在L-Buffer的基础上重新渲染场景并进行最终的shading
- [deferred texturing](../GPUDrivenRenderPipeline_references/DeferredTexturing/DeferredTexturing.md):三个pass。纹理采样等计算都放到最后一个pass去处理，生成G-Buffer Pass只保存UV、材质ID、顶点法线等信息
  - 生成Gbuffer
  - Light-Culling Pass
  - shading 阶段

刺客信条的分享上面参考的是deferred texturing。

# deferring texturing实现记录
未添加各种粗粒度的剔除。仅靠frustrum进行剔除。

## 开始记录的状态

<div align="center">

![StatusBefore][StatusBefore]

</div>

## G-Buffer的结构
按照参考blog中G-Buffer需要记录的基本结构，设计G-Buffer的结构为：

|index | float0 | float1| float2| float3|
|:---:|:---:|:---:|:---:|:---:|
|RT0 | |Depth  ||Stencil |
|RT1|Normal 2x16 定点数|UV 2X16 定点数| Mat ID |

## MRT
用来是想G-Buffer的填充。[官方文档](https://docs.microsoft.com/en-us/windows/win32/direct3d9/multiple-render-targets)。

> 优缺点：
> - 所有RT必须使用相同的Depth, 但是可以有不同的格式
> - 宽高必现相同
> - 有些后处理不能现实，比如：抖动，透明度测试，雾，混合或者masking。z-test和stecencil test可以。
>   - 设置了D3DPMISCCAPS_MRTPOSTPIXELSHADERBLENDING 之后，可以进行Alpha Blend、Alpha Test、Fog等操作，但是有他们自己的限制。
> - 不能AA
> - 输出写遮罩需要视不同的硬件决定



## 填充G-Buffer的pass
- 使用MRT技术，两个RT使用不同的Format。
- 打开Stencil为后面的shading使用

具体实现：
- G-Buffer包含2个RT
- 2个RT：2个资源对应2个Tex View和两个RenderTargetView
- 单独的PSO：设置RT数量和独立的RT format
- OMSetRenderTarget设置RenderTarget的数量和目标
- 调试可以把RenderTarget给复制到BackBuffer中进行调试

## 进行Shading的pass
### code和decode
- normal:需要考虑2个通道表示3通道之后的Z方向问题、边界问题等，可以参考[网址](https://www.xuebuyuan.com/439585.html)
- tangent:
- uv:
- depth:用来在PS中反算世界坐标

这里的精度都是把float3 float2压缩到16X2的uint定点数中，针对float的24位精度，会存在精度损失。

## 延迟渲染的优化
主要就是要减少带宽，把消耗高的计算推迟到后面。

做了如下的努力：
- 使用Deferred Texturing
- 使用定点数

[StatusBefore]: ./StatusBefore.png