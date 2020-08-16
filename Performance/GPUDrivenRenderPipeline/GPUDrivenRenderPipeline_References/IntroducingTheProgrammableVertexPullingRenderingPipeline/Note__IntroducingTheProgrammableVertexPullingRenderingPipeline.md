<!-- TOC -->

- [1. 简介](#1-简介)
- [理论](#理论)
  - [Programmable Draw Dispatch](#programmable-draw-dispatch)
  - [Programmable Vertex Fetching](#programmable-vertex-fetching)
- [软件设计上的细节处理](#软件设计上的细节处理)
  - [如何达到图元峰值处理速率](#如何达到图元峰值处理速率)
  - [内存重新打包](#内存重新打包)
- [展望](#展望)

<!-- /TOC -->

[原文](./GPU%20Pro%204.pdf)

# 1. 简介
基本是背景介绍，参看原文。

三种基本的减少draw call的方式：
- instancing:一个mesh重复多次
- SharedVBO：使用Vertex Buffer数组
- 独立的VBO：最传统的模式

效率：instancing>SharedVBO>独立的VBO

实际上是CPU负载的一种反应，资源跟环境切换越多，则draw call越多。

最可行的模式是在instancing和SharedVBO模式之间寻找一个平衡点。

>CPU负载的实质：
> - 提交命令时有个验证步骤
> - 关乎到根据顶点设定进行顶点组装的设置阶段

减少CPU负载的方式：

减少资源等的切换，把资源进行打包提交。

- 对于VAO，打包共享相同顶点格式/基址的多个mesh以在每个draw的时候访问正确的数据
- 对于纹理，可以使用纹理数组
- 对于uniform内存，可以根据更新频率打包到更大的uniform buffer中

# 理论
Programmable Vertex Pulling Rendering Pipeline:把culling和sorting从CPU移到GPU上。

主要分为两个阶段：
- Programmable Draw Dispatch
- Programmable Vertex Fectching

work flow :

<div align="center">

![][PVPRPWorkFlow]

</div>

## Programmable Draw Dispatch
- 在GPU上生成draw paramenters来填充Indirect Draw Buffer
- API中用到的DrawCount不要从上面生成的Indirect Draw Buffer中来获取，因为这样会导致同步问题，而应该使用Indirect Draw Buffer中的参数来做图元的剔除

## Programmable Vertex Fetching
为了降低CPU的负载，不再使用VAO（需要顶点格式一样），直接在GPU中组装顶点而不用考虑通过顶点来sort draws。

这种处理存在一个问题，没法准确定义DrawID
    - 如果使用VertexID == 0来判断的话，对顶点的操作不是原子的，没法准确进行判断
    - 存储于一个32位的顶点属性中。使用BaseInstance和一个等于1的属性来判断，这个属性被一次draw中的所有顶点调用所共享。DrawID的部分位用来编码顶点格式ID，剩下的用来索引一次draw的第一个顶点

# 软件设计上的细节处理
## 如何达到图元峰值处理速率
通过实验方式。找到这个值之后，就有机会能在保证达到了峰值速率之后对复杂度不高的三角形进行细分等处理。
> 细分既可以添加几何复杂性，也可以用来保证图元处理速率。

## 内存重新打包
实体创建和删除之后导致的内存碎片处理：
- glCopyBufferSubData和glCopyImageSubData
- 使用OpenCL Kernel来移动内存
- 使用Virtual Memory

# 展望
有点儿东西，可以参看原文。

[PVPRPWorkFlow]: ./PVPRPWorkFlow.jpg
