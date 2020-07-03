
> 主要为[Houdini 12 标准教程](https://www.zhihu.com/pub/book/119589701)的学习记录

# 主要模块
## work flow

<div align="center">

![模块][Modules]

</div>

## 节点

<div align="center">

![模块][Nodes]

</div>

- 功能的实现靠功能下的节点，关系类似于Unix的文件夹结构
- 模块通过节点路径切换

# UI
## paramenters 和 attributes的区别
- paramenters 对应的具体的参数、UI交互接口等
- attributes 对应的是geometry相关的数据
- 可以理解为paramenter 是对 attribute 进行操作所暴露的接口

# UV
- UDIMS：一种平铺Texture和UV的技术，让一张大的纹理平铺到多个tile上。

# Rendering
- MANTRA: Scanline, Raytracing, PBR.

# Expressions & Scripting
- Hscript
- Python
- VEX

[Modules]: ./HoudiniModules.jpg
[Nodes]: ./HoudiniNodes.jpg