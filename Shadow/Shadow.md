- [1. 基本实现原理](#1-基本实现原理)
    - [1.1. 平行光阴影](#11-平行光阴影)
    - [1.2. 级联阴影——Cascade Shadows](#12-级联阴影cascade-shadows)
    - [1.3. 不足](#13-不足)
- [2. 细节优化](#2-细节优化)
    - [2.1. Shadow Acne 问题](#21-shadow-acne-问题)
    - [2.2. 边缘阴影闪烁](#22-边缘阴影闪烁)
    - [2.3. 软阴影处理](#23-软阴影处理)
    - [2.4. 剔除](#24-剔除)

# 1. 基本实现原理
目前的阴影的实现方式主要有两种方式：Shadow Map 和 Shadow Volume。Unity采用Shadow Map，本篇主要介绍的是Showdow Map的实现方式。

针对单一光源，Shadow Map的实现原理，实际上是通过一张texture记录光源在当前屏幕坐标系下的阴影信息。texture中一个像素采用单通道的方式，记录了在光源坐标系下，屏幕上对应的fragment到光源的距离，在实际渲染过程中，计算fragment在光源坐标系中到光源的距离与ShadowMap采样结果进行比较，判断是否在该光源的阴影中，然后进行后续的判断。

针对多光源的处理，就是单个光源处理的loop。
## 1.1. 平行光阴影
平行光光源位置在无限远处，在u3d中通过统一的接口SAMPLE_TEXTURE2D_SHADOW处理了fragment到光源的距离。

采用正交投影。
>SAMPLE_TEXTURE2D_SHADOW 通过fragment在光源坐标系中的位置获取当前的ShadowAttenuation来计算当前的fragment的阴影。
        
## 1.2. 级联阴影——Cascade Shadows
场景中的相机采用透视投影，平行光阴影使用的是正交投影，意味着在屏幕坐标系中，离场景相机越远的fragment越容易采样到同一个Shadow Map的同一个像素值，也即一个texel要覆盖更多的fragment，这就会导致阴影锯齿、太小的fragment shadow容易丢失以及**Shadow Acne**等问题。

级联阴影，通过将光源的光照范围进行分级，每级生成不同的Shadow Map，保证在边缘的fragment能和在较近的fragment有一个接近的表现。

<div align=center>

![][CasadeShadowMap]

Casade Shadow Map

![][CasadeShadowsCull]

Casade Shadow Cull
</div>

## 1.3. 不足
1. 一个光源就需要一个ShadowMap,多个光源需要多个ShadowMap。虽现在的场景一般都会限制光源数量，但是为了为了节省空间，一般还是会使用一个Altas来将多个光源的ShadowMap存储在一个texture中，这样实际上会降低采样的精度，导致**Shadow Acne**问题。

2. 不是场景中所有的物体进行阴影绘制，除了有Shadow Caster和Receive Shadow设置的实体，场景中还通过Shadow Max Distance和*级联阴影Cascade Shadow*进行剔除了操作。

3. 级联阴影的剔除是使用包围球，但是屏幕坐标系实际上是一个矩形，因此会出现在包围盒周围的阴影可能会存在闪烁现象，通过设置**Bias**可以减轻这种情况。

4. 级联阴影将一个光源的Shadow Map拆分为不同的层级的多个Shadow Map,虽然提高了离光源较远的Shadow Map精度，但是在Shadow Map Altas尺寸的不变的情况下，在整体上降低了所有Shadow Map的精度，会加剧**Shadow Acne**问题。
        
# 2. 细节优化
## 2.1. Shadow Acne 问题
    
导致Shadow Acne问题的主要原因是Shadow Map 的一个texel要照顾到屏幕坐标系中的多个fragment，再加上Shadow Map Altas的精度不够大，SAMPLE_TEXTURE2D_SHADOW在计算时会损失掉一些精度，导致相邻的fragment会取Shadow Map Altas上的同一个texel值。

解决的方案主要有：增加Shadow Map Altas的尺寸，但是一般对texture的最大尺寸都有限制；在生成Shadow Map Altas的时候添加Depth Bias;在采样Shadow Map Altas的时候添加Normal bias。要得到一个理想的结果，往往是好几个参数一起联调的结果。

## 2.2. 边缘阴影闪烁
 同[剔除中的Max distance](#24-剔除)

## 2.3. 软阴影处理

1. Fading Cascades 
    
    在Cascade Shadow边缘，阴影都是突然截断，采用比较柔和的方式来实现阴影的渐隐。

    这里使用的公式$\frac{1-\frac{d^2}{r^2}}{f}$，其中d表示当前fragment到光源的距离，r表示当前Cascade剔除球的半径，f表示一个修正的系数。把这个公式计算得到的值用于Shadow Auttention的修正。

2. PCF Filtering
    
    类似模糊后处理效果，通过采样当前fragment对应的Shadow Map附近的几个像素值来获得一个平均值作为当前的采样结果，让阴影变化更加柔和。

3. Blending Cascades 

    在Cascade边缘附近的fragment影子容易出现切割，所以需要取当前的Cascade 和下一个Cascade进行混合，保证Cascade边缘的阴影柔和变化。

4. Dithered Transition

    Blending Cascades 需要在两个Cascade 中采样，有多余的性能消耗。Dithered Transition采用抖动的方式，概率性的取当前Cascade或者下一个Cascade进行采样，虽然整体表现差不多，但是再放大之后的fragment表现差别还是差很多，具体参见参考的blog。

## 2.4. 剔除
1. Max distance

进行阴影剔除的最初方式，大于当前distance的fragment不用进行阴影的相关判断。

但是如果Max distance 太小，小于Cascade最大的半径，再加上精度问题就会出现阴影闪烁的情况。

2. Culling Bias

直接设置阴影剔除的参数。

[CasadeShadowsCull]: ./culling-spheres.png
[CasadeShadowMap]: ./one-light-four-cascades.png

