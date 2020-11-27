<!-- TOC -->

- [1. 简介](#1-简介)
- [2. 基本语法](#2-基本语法)
- [3. 纹理数组](#3-纹理数组)
- [4. 纹理子资源](#4-纹理子资源)
- [5. alpha-to-coverage 技术](#5-alpha-to-coverage-技术)
- [6. 细节问题记录](#6-细节问题记录)
- [7. 实践细节](#7-实践细节)

<!-- /TOC -->

# 1. 简介
- 没有细分着色器的话，可以放到顶点和像素着色器之间
- 以每个三角形的3个定点作为输入，输出对应的图元列表
- 可以创建或者销毁图元，也可以改变图元类型

# 2. 基本语法
- maxvertexcount：性能跟标量数量相关
- 输入参数：
- 输出参数：

# 3. 纹理数组
Texture2DArray。
不使用如下表达方式的原因：
- Shader Model 5.1之前不行
- 产生多余开销

加载：
- 打包成一个图集dds

# 4. 纹理子资源
- 数组切片：纹理数组中的某个纹理及其mipmap链。
- mip切片：纹理数组中特定层级的所有mipmap。
- 子资源：纹理数组中某纹理的单个mipmap层级。

<div align="center">

![][TextureSubRes]

![][TextureSubResIndice]

</div>

# 5. alpha-to-coverage 技术
前提：
- 开启了MSAA

原理：
根据PS返回的alpha值，来判断有几个子像素在多边形之外。比如，在4x MSAA时，如果PS返回的alpha值为0.5， 则可以认为4个像素中的2个，在多边形之外。

# 6. 细节问题记录
- 每次DrawIndexInstanced/DrawInstanced的时候，vertex_id都会从0开始计算。

# 7. 实践细节
- 编写和编译GeometryShader
- 绑定到PSO上

[TextureSubRes]: ./TextureSubRes.bmp
[TextureSubResIndice]: ./TextureSubResIndices.bmp
