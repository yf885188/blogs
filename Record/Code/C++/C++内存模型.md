# 参考
[链接](https://www.bilibili.com/video/BV1Kb411B7N8?p=1)

# 内存分配的4个层次

<div align="center">

![][MemoryHierarchy]

![][MemoryHierarchy1]

</div>

用户态的实现下，内存分配最终都回到了对malloc和free的调用上。

[MemoryHierarchy]: ./MemoryHierarchy.jpg
[MemoryHierarchy1]: ./MemoryHierarchy1.jpg

# new 和 delete
## new
编译器的实现过程：
- 分配内存
  - 内存大小
  - 根据类形成内存布局
- 调用构造函数

## delete
编译器的实现过程：
- 调用析构函数
- free内存

# Array new
<div align="center">

![][ArrayNewMemStructure]

</div>

- 对于array如果使用的是delete而不是delete[],则会因为访问到的是数组的个数，而不是实例地址，导致错误。
- 基本类型不会有这样的问题。

[ArrayNewMemStructure]: ./ArrayNewMemStructure.jpg

# Placement New

<div align="center">

![][PlacementNewWorkFlow]

</div>

## work flow
- 定位内存
- static_cast 转变内存类型
- 调用构造

[PlacementNewWorkFlow]: ./PlacementNewWorkFlow.jpg