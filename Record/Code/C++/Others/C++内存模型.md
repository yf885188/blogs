<!-- TOC -->

- [参考](#参考)
- [内存分配的4个层次](#内存分配的4个层次)
- [new 和 delete](#new-和-delete)
  - [new](#new)
  - [delete](#delete)
- [Array new](#array-new)
- [Placement New](#placement-new)
  - [work flow](#work-flow)
- [G2.9 std::alloc的运行模式](#g29-stdalloc的运行模式)
- [malloc/free](#mallocfree)
- [bitmap_allocator](#bitmap_allocator)

<!-- /TOC -->

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

- placement delete 只有在对应的placement new 发生异常的时候才会被调用。

## work flow
- 定位内存
- static_cast 转变内存类型
- 调用构造



[PlacementNewWorkFlow]: ./PlacementNewWorkFlow.jpg

# G2.9 std::alloc的运行模式
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

# malloc/free

VC6 CRT Start Up Flow:

<div align="center">

![][CRTStartUp]

</div>

- 上下cookie可以用于合并时的条件检测
- 根据Group中记录的count来判断当前链表是否是空，是否能全回收
- 只有在有两个以上的Group需要全回收的时候才进行全回收，也即defering——延迟机制
  - 有一个Group要回收，记为defer
  - 等第二个Group要回收，把defer Group回收，然后把第二个触发全回收的Group记为defer

[CRTStartUp]: ./CRTStartUp.jpg


# bitmap_allocator

<div align="center">

![][BitmapAllocatorStructure]

</div>

- super-blocks = blocks + bitmap。
- 若没有全回收，则分配时N+1级的super-block的size大小是N级的super-block的size大小的2倍。类似容器的容积2倍的增长。反之，分配时的size会减半。
- 同一super-block内只会分配同一类型的block，即使block size相同也是如此。
- 全回收的super-block会被放到一个free list，并按照super block的大小进行排列。如果这个free list的大小达到64的上限，新回收的super-block如果大于free list尾部最大的super block size会被直接delete，反之替代掉尾部的super block。
- 如果所有的super block都进到free list中，此时又触发了block的分配，就直接从free list中选取一个super block回来接着使用。

[BitmapAllocatorStructure]: ./BitmapAllocatorStructure.jpg