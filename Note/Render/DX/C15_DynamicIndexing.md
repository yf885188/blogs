<!-- TOC -->

- [1. 动态绑定](#1-动态绑定)

<!-- /TOC -->

# 1. 动态绑定
在GPU端能对资源进行动态索引，model 5.1以上的功能。

> 与之前相比：
> - 之前绑定数组内资源，需要在CPU端，根据Heap的其实位置和描述符的跨度，以及当前资源的索引，计算偏移位置之后进行绑定。
> - 使用动态绑定之后则不用在CPU端一个个绑定，一次性绑定该资源数组的起始位置，之后在GPU端的shader内通过索引获取相关资源。