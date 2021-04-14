# 架构
## 粗分
- 应用层
- 几何化过程
- 光栅化
- 像素化过程

### 应用层
CPU端，任务：
- 碰撞检测
- 全局加速算法
- 动画
- 物理模拟
- ...

最主要的任务，就是为下一个阶段筛选需要的绘制的prim，或者图元。

为了加强这一阶段的性能，通常把这一阶段的表现并行在几个处理核心上进行计算。

### 几何化过程
GPU端，任务：
- 变换
- 投影
- ...

处理大部分的逐面片和逐顶点操作。

进一步划分如下图所示的流程：

![][GeometryProcessing]

[GeometryProcessing]: ./images/GeometryProcessing.png

#### 顶点着色 Vertex Shading
两个主要任务：
- 计算顶点位置
- 计算VS的输出：normal、uv等等

各种变换：
model transform -> view transform -> projection

两种常用投影：
- orthographic
- perspective

##### 可选的顶点过程
按照顺序有：
- 细分过程：包含hull shader，tessellator和domain shader
- 几何着色
- 流式输出：把结果传到一个buffer做备用，而不是传给后面的阶段

#### Clipping
通过投影产生的4维齐次坐标来做clip。

因为在投影空间无法进行线性插值，所以需要第四个维度的参数进行插值。然后进行投影除法，得到NDC坐标，最后通过NDC变换到窗口坐标系。

#### Screen Mapping
刚进入这个阶段的时候，坐标还都是3D的。

> 屏幕坐标和窗口坐标的区别：
> - 屏幕坐标：(x,y)
> - 窗口坐标：(x,y,z)

Screen Mapping 就是把不同窗口坐标映射到对应的窗口。[-1, 1] -> [w,h]范围的映射。

> opengl和DX的坐标系区别
> - Opengl:左下角原点
> - DX:左上角原点


### 光栅化
GPU端，任务：接受3个顶点组成的三角形，判断他们覆盖的pixels，然后传到下一个阶段。

### 像素化过程
GPU端，任务：逐像素的着色，以及深度测试、混合等等操作。

