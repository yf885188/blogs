# Shadow
[toc]

# 基本实现原理
目前的阴影的实现方式主要有两种方式：Shadow Map 和 Shadow Volume。Unity采用Shadow Map，本篇主要介绍的是Showdow Map的实现方式。

针对单一光源，Shadow Map的实现原理，实际上是通过一张texture记录光源在当前屏幕坐标系下的阴影信息。texture中一个像素采用单通道的方式，记录了在光源坐标系下，屏幕上对应的fragment到光源的距离，在实际渲染过程中，计算fragment在光源坐标系中到光源的距离与ShadowMap采样结果进行比较，判断是否在该光源的阴影中，然后进行后续的判断。

针对多光源的处理，就是单个光源处理的loop。
## 平行光阴影
平行光光源位置在无限远处，在u3d中通过统一的接口SAMPLE_TEXTURE2D_SHADOW处理了fragment到光源的距离。

采用正交投影。
>SAMPLE_TEXTURE2D_SHADOW 通过fragment在光源坐标系中的位置获取当前的ShadowAttenuation来计算当前的fragment的阴影。
        
## 级联阴影——Cascade Shadows
场景中的相机采用透视投影，平行光阴影使用的是正交投影，意味着在屏幕坐标系中，离场景相机越远的fragment越容易采样到同一个Shadow Map的同一个像素值，也即一个texel要覆盖更多的fragment，这就会导致阴影锯齿、太小的fragment shadow容易丢失以及**Shadow Acne**等问题。
级联阴影，通过将光源的光照范围进行分级，每级生成不同的Shadow Map，保证在边缘的fragment能和在较近的fragment有一个接近的表现。

<div align=center>

![][CasadeShadowMap]

Casade Shadow Map

![][CasadeShadowsCull]

Casade Shadow Cull
</div>

## 不足
1. 一个光源就需要一个ShadowMap,多个光源需要多个ShadowMap。虽现在的场景一般都会限制光源数量，但是为了为了节省空间，一般还是会使用一个Altas来将多个光源的ShadowMap存储在一个texture中，这样实际上会降低采样的精度，导致**Shadow Acne**问题。

2. 不是场景中所有的物体进行阴影绘制，除了有Shadow Caster和Receive Shadow设置的实体，场景中还通过Shadow Max Distance和*级联阴影Cascade Shadow*进行剔除了操作。

3. 级联阴影的剔除是使用包围球，但是屏幕坐标系实际上是一个矩形，因此会出现在包围盒周围的阴影可能会存在闪烁现象，通过设置**Bias**可以减轻这种情况。

4. 级联阴影将一个光源的Shadow Map拆分为不同的层级的多个Shadow Map,虽然提高了离光源较远的Shadow Map精度，但是在Shadow Map Altas尺寸的不变的情况下，在整体上降低了所有Shadow Map的精度，会加剧**Shadow Acne**问题。
        
# 细节优化


[CasadeShadowsCull]: ./culling-spheres.png
[CasadeShadowMap]: ./one-light-four-cascades.png