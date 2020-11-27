
[参考](https://zhuanlan.zhihu.com/p/135444145)

# 前项渲染中的MSAA流程
work flow

<div align="center">

![][MSAAWorkFlow]

</div>

[MSAAWorkFlow]: ./MSAAWorkFlow.jpg

# DS不支持MSAA的原因
- GBuffer填充之后，在shading阶段，不知道三角形的覆盖信息，也即无法计算coverage。
- 如果使用多倍的GBuffer，消耗变大不说，原始像素点的位置深度、法线等几何信息又都是通过顶点信息插值出来的，这个时候也丢失了。

综上，在GBuffer填充之后，丢失了相关的几何信息，无法进行MSAA

# 改进DS支持MSAA
在填充BaseColor的时候进行MSAA
> 但是对于很多改进之后的DS算法，比如deferred texturing这种延迟渲染的方式，BaseColor的填充是在最后的shading阶段，这种方式的处理仍然无法使用MSAA。
