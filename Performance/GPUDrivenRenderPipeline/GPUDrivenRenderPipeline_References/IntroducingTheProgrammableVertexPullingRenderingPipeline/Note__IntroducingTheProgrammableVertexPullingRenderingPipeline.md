<!-- TOC -->

- [1. 简介](#1-简介)

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