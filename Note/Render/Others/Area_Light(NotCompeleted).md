[参考](https://zhuanlan.zhihu.com/p/81564617) 以及PBR C12

# 原理
简化的做法是将面光源所处的空间划分为9个区域，在不同的区域面光源退化为点光源或者线光源。

这种方式，规避了积分的运算，但是也很不PBR。

PBR的方式是对面光源进行积分。为了快速的计算，针对不同的面光源类型进行了不同的trick处理。

# LCT
LCT 是SIGGRAPH2016上提出来的一种平面面光源的处理方式。[参考](https://eheitzresearch.wordpress.com/415-2/)

