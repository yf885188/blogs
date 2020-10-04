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

## G2.9 std::alloc的运行模式
去掉单次内存分配时cookie等带来的内存消耗。

work flow:

<div align="center">

![][AllocWorkFlow]

</div>

- 16个链表负责16个不同大小的内存链表，超过这16个尺寸的内存分配则直接调用malloc。
- 每次分配内存时先看备用内存是否存在
  - 不存在：申请2*20个size大小内存。多余的20个size用于备用。
  - 存在：
    - 大小够用：直接使用备用的内存，备用的内存不一定够20个size，但是满足当前分配的数量也算够用。
    - 大小不够：把备用内存挂在响应的内存链表下，然后申请新的内存，也即跳刀不存在备用内存池的步骤。
- 如果所有内存都被分配到了各自大小的内存链表中，这时再分配内存就会导致内存不足，此时会从右边最靠近的内存链表中获取相关的内存作为备用内存池，然后分配给当前的内存链表。如果右边没有则会导致分配失败。

[AllocWorkFlow]: ./AllocWorkFlow.jpg