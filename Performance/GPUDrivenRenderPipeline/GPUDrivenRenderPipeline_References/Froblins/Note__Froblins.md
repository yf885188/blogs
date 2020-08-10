<!-- TOC -->

- [1. 论文目的](#1-论文目的)
- [2. 主要解决的问题](#2-主要解决的问题)
- [3. 动态寻路](#3-动态寻路)
  - [3.1. 全局寻路](#31-全局寻路)
    - [3.1.1. GPU上的寻路](#311-gpu上的寻路)
    - [3.1.2. Slover：从价值到旅行时间](#312-slover从价值到旅行时间)
      - [3.1.2.1. 基于快速匹配方法的Eikonal Solver](#3121-基于快速匹配方法的eikonal-solver)
      - [3.1.2.2. 并行的Eikonal Solver](#3122-并行的eikonal-solver)
      - [3.1.2.3. 把环境抽象成价值函数](#3123-把环境抽象成价值函数)
  - [3.2. 局部导航和避障](#32-局部导航和避障)
    - [3.2.1. 方向影响](#321-方向影响)
      - [3.2.1.1. 空间查询](#3211-空间查询)
- [4. 光照](#4-光照)
  - [4.1. 球谐函数光照纹理](#41-球谐函数光照纹理)
  - [4.2. Double Shadow问题](#42-double-shadow问题)
- [5. GPU的场景管理](#5-gpu的场景管理)
  - [5.1. 使用几何着色器做过滤](#51-使用几何着色器做过滤)
  - [5.2. work flow](#52-work-flow)
    - [5.2.1. Hi-Z map](#521-hi-z-map)
    - [5.2.2. 用于LOD选择的GS过滤](#522-用于lod选择的gs过滤)
  - [5.3. 围绕Queries来组织Draw Calls](#53-围绕queries来组织draw-calls)
- [6. 动画模拟](#6-动画模拟)
- [7. 细分](#7-细分)
- [8. PrePass 和 顶点压缩](#8-prepass-和-顶点压缩)
- [9. 地形绘制](#9-地形绘制)

<!-- /TOC -->

[原文](./../References/Oat-Tatarchuk-Froblins(Siggraph2008).pdf)

# 1. 论文目的
在GPU上模拟和渲染大量角色。

# 2. 主要解决的问题
- GPU上的动态寻路
- LOD的群体渲染
- 在保证高质量和稳定的特写镜头下做细分
- HDR光照和使用正确gamma渲染的后处理效果
- 地形系统
- 适用于大范围场景的级联阴影
- 改进的全局光照系统

# 3. 动态寻路
- 把全局的寻路与局部避障和个体选择结合起来
- 群体的移动在全局看来更加的稳定和真实
  - 群体的动态表现跟液体类似
  - agents会沿着有最小阻力的路径朝最近的目标移动
  - 角色会跟着正确的道路并且不会阻塞

## 3.1. 全局寻路
- 基于连续人群
- 把运动规划问题转换成一个优化问题
  - 使用连续模型
  - 光学计算算法
- 平滑流动类似的群体移动
  - 避障
  - 合并

### 3.1.1. GPU上的寻路
- 使用迭代计算的方式（eikonal solver）在GPU上求解一个2阶微分方程
  - 把环境当成价值值域
  - 通过离散的eikonal公式求解
- 适用于很多算法和领域

### 3.1.2. Slover：从价值到旅行时间
- 求解旅行时间作为势能的函数
- 势能=沿着最短路径到达目标的综合成本
  - 沿着负梯度方向
- 设置目标点的势能为0，其他地方用eikonal方程计算

#### 3.1.2.1. 基于快速匹配方法的Eikonal Solver
- 仿真Dijkstra的最短路径算法
- 对于连续eikonal公式的有限差分近似处理
  - 从已知的势能点（目标点的零势能点）向周围传播直至收敛
  - 基于已知的cell计算周围cell的势能
- 串行算法：需要一个有序的数据结构

#### 3.1.2.2. 并行的Eikonal Solver
- 使用快速迭代方法
- 使用逆离散差分近似，不需要有序的数据结构
- 使用低的grid规模
- 一次计算4个目标
  - 利用向量化的优势
  - 使用RGBA FP16格式的纹理贴图
- 一种经验式的方式进行收敛

#### 3.1.2.3. 把环境抽象成价值函数
- 连续模型把环境建模成一个正向的价值函数
  - 不能为0， 否则其他的agents会使用一种离散不连续的速度运动。增加一定基础量。
- 把地形和（静态和动态的）障碍物的信息都包含进来

## 3.2. 局部导航和避障
- 全局模型对剧本障碍物不太适用
  - 表现：没必要远离agent
- 根据附近agents/障碍物的速度和位置来更新速度
  - Based on Velocity Obstacle formulation [Fiorini and Shiller 1998]
  - 参考： Jeremy Shopf’s talk on in Beyond Programmable Shading: In Action course

### 3.2.1. 方向影响
- 相对于全局导航方向，评估一组固定的可能方向集合

#### 3.2.1.1. 空间查询
- 查询附近agent的位置和速度
- 空间数据结构
- 通过位置剔除agents
  - 剔除数量：color buffer
  - 剔除数组：depth buffer array

<div align="center">

![数据结构][Bins]
数据结构

![work flow][BinQuery]
work flow

![update][BinUpdate]
Bin Update(细节参看原文)

</div>

> 注意：
> - 迭代的提前结束
> - 避免同步导致的阻塞

# 4. 光照
## 4.1. 球谐函数光照纹理

<div center="align">

![][DiffuseLightingWithShadowMap]

</div>

## 4.2. Double Shadow问题
现象：在阴影中的实体也释放阴影了。
解决：阴影地图只用于没有遮挡的地形区域。（原文通过地形的亮度值变化来判断是否被遮挡）

<div center="align">

![][DetectingDirectSunLight]

</div>

# 5. GPU的场景管理
使用GPU做场景管理的原因：
- 需要可扩展的和稳定的表现
- 不想要渲染成千上万面的角色
- 用GPU做模拟的时候，不能在CPU端进行角色管理，因为需要一个read-back。

## 5.1. 使用几何着色器做过滤
- 基于instance
- 把instance的顶点作为输入
- 通过了特殊测试的顶点才能被输出，其他的都被丢弃
  - 使用DrawAuto做多个测试，或者合并到一个GS中做测试来提高效率。

## 5.2. work flow
<div align="center">

![][GPUSceneManagementWorkFlow]

</div>

### 5.2.1. Hi-Z map
- 一个层级深度图
  - 使用Zbuffer的信息
- 一个mip-mappe全屏尺寸图片
- 不需要一个独立的depth pass

关于Hi-Z map会在另一篇做详细的总结。
- 使用Hi-Z Map做遮挡剔除
- Hi-Z Map的构建
- Hi-Z Levels的Reduction Pass

### 5.2.2. 用于LOD选择的GS过滤
- 使用离散的LOD构建
- 三个接替执行的filtering pass
  - 把角色分割到3个离散的集合中
  - 可以很容易的为每个集合指定LOD参数
- 在后剔除的操作做了之后在计算LOD的选择
  - 只执行可见的角色
  - 剔除的结果仅仅计算一次并且能重复使用
- 使用细分和Displacement渲染最近的LOD
- 常规方式渲染中间的LOD
- 最简单的方式渲染最远的LOD

## 5.3. 围绕Queries来组织Draw Calls
- 需要知道实体数量来为每个LOD分配draw call
- 需要一个stream来返回状态query
  - 如果在查询query的当前帧使用这个query会导致明显的阻塞
- 在查询query和使用query结果之间重新组织draw calls来填充gap

# 6. 动画模拟
这个之前已经实践过了，[参考](https://github.com/529038378/GPU_Instancing_demo)

# 7. 细分

# 8. PrePass 和 顶点压缩

# 9. 地形绘制
- 顺滑无裂纹无须退化的LOD
- 细分和实例化
- 使用DX10.1的特性减少了内存占用
- 复杂的材质系统


[Bins]: ./Bins.png
[BinQuery]: ./BinQuery.png
[BinUpdate]: ./BinUpdate.png
[DiffuseLightingWithShadowMap]: ./DiffuseLightingWithShadowMap.png
[DetectingDirectSunLight]: ./DetectingDirectSunLight.png
[GPUSceneManagementWorkFlow]: ./GPUSceneManagementWorkFlow.png

