<!-- TOC -->

- [1. 优势](#1-优势)
- [2. 定义](#2-定义)
- [3. 实现原理](#3-实现原理)
- [4. work flow](#4-work-flow)
- [5. 存在的问题](#5-存在的问题)
- [6. 扩展](#6-扩展)

<!-- /TOC -->

[原文](http://www.reedbeta.com/blog/deferred-texturing/)

# 1. 优势
- 把lighting code和材质code分离
- 每帧只处理一次场景中的几何体
- 减少之后被遮挡剔除的不必要的着色工作
- 减少发生在读、写和修改屏幕缓冲上的带宽消耗

# 2. 定义
只有在知道屏幕上最终是啥结果之后，再去执行纹理采样和材质计算。

# 3. 实现原理
- 为了实现延迟采样纹理：在G-Buffer里存储UV和材质ID
- 注意mipmap和各项异性过滤的处理，不要跨过材质边界或者UV缝合处
- 在G-Buffer中保存插值后的顶点法线数据

所以G-Buffer的结构应该如下：
- 顶点法线：2x16-bit 定点数
- 材质ID： 16-bit int
- UVs：2x16-bit [0,1]范围或者[0,2]范围的定点数，或许考虑到更大的虚拟纹理需要32-bit
- 可能：16-bit float 的LOD或者4x16-bit float的UV derivatives

# 4. work flow
- 渲染场景，生成如上的G-buffers
- 做一个light-culling pass
- 在屏幕空间为每个像素做如下处理：
  - 查找材质信息
  - 使用bindless texturing来为像素当前的材质中的所有纹理进行采样
  - 做必要材质计算
  - 做光照计算

# 5. 存在的问题
- 所有纹理通过bindless的方式加载到GPU，这里需要考虑硬件是否已经支持
  - 可以采用一种平铺策略：在compute shader中将屏幕平铺中所有展示的材质收集起来，然后遍历这些材质并一起对每个纹理进行材质查找。
- 如果你是用数组的方式存储纹理的话，甚至不需要通过材质ID进行纹理查找，仅仅通过size/format组合就能用来决定使用哪个纹理组。每个像素都能使用不同的纹理索引，除了缓冲的局部性以外，基本不会有其他后果。
- 当然，也能使用将所有纹理都存储于一个大的图集中。这样的话，会牺牲掉硬件UV寻址模式，也需要所有的纹理都是同一种格式。
- 也可以按照材质进行处理，每次只处理屏幕空间中当前材质的像素。这个比第一种方式有更好的缓存局部性也更快。

强制使用Bindless纹理有巨大的风险：
- 有LOD或者derivatives的纹理采样相对于自动的LOD机制会变慢
- 材质代码和光照代码在一个巨大的shader中，会导致一系列问题：编译，占用和分支处理等等
- 每个图元只能有一个UV。如果其他的纹理采样公用一套UV或者根据这套UV进行变化得到的话还好，但是碰到像使用lightmaps这种需要一套独立UV的情况，会存在问题
- 和deferred shading类似，deferred texturing对MSAA不友好。
- 跟deferred shading相比，deferred texturing对透明的处理更加不友好。

# 6. 扩展
https://www.slideshare.net/CassEveritt/approaching-zero-driver-overhead
