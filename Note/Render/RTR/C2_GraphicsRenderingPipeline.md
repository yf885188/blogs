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

[GeometryProcessing.png]: ./images/GeometryProcessing.png

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

### 光栅化
GPU端，任务：接受3个顶点组成的三角形，判断他们覆盖的pixels，然后传到下一个阶段。

### 像素化过程
GPU端，任务：逐像素的着色，以及深度测试、混合等等操作。

