# 空间配置器的标准接口
![][AlocatorStandardInterface]

[AlocatorStandardInterface]: ./AlocatorStandardInterface.jpg

# SGI Allocator
具有sub-alloctaion能力的SGI Allocator。

- std::alloctor:对::operator new和::operator delete的浅层包装
- std::alloc

文件结构：
![][AlocatorFileSturture]

[AlocatorFileSturture]: ./AlocatorFileSturture.jpg

## constructor和destructor
![][AllocatorInConstructorAndDestructor]

[AllocatorInConstructorAndDestructor]: ./AllocatorInConstructorAndDestructor.jpg

> 在使用迭代器对容器进行destroy的时候，会判断当前类的析构函数是否trivial，如果是trivial的话，直接不操作。反之，则按照迭代器遍历进行析构。

# 内存管理
SGI的原则：
- 向system heap要求空间
- 考虑多线程状态
- 考虑内存不足时的应变措施
- 考虑过多“小型区块”可能造成的内存碎片问题

为了解决碎片问题，SGI设置了双层级配置器
- 第一级配置器直接使用malloc和free
- 第二级视情况采用不同的的策略：
  - 配置区块超过128bytes，视为足够大，调用第一级配置器
  - 反之，使用memory pool方式

具体的可以看[视频](https://www.bilibili.com/video/BV1Kb411B7N8)

![][SGIAllocLevels]

[SGIAllocLevels]: ./SGIAllocLevels.jpg

内存不足时候的分配的优先级：
- 满足多个size（不满足全部）的内存块
- 一个size都无法满足，把当前free list的这个多余区块进行重新分配
- 向heap请求内存
- free list不同规格进行遍历，查看有无free list并进行处理
- 调用out-of-memory机制

# 内存基本处理工具
五个全局函数：
- construct
- destroy
- uninitialized_copy:对应于copy
- uninitialize_fill：对应于fill
- uninitialize_fill_n：对应于fill_n

> - 会根据是否是POD来采用不同的处理机制：如果是POD直接使用数值填充这种最效率的方式；反之使用构造函数这种安全的方式。
> - 针对char* / wchar_t*类型，最优效率的做法是使用memmove。

![][UnitializeFuncs]

[UnitializeFuncs]: ./UnitializeFuncs.jpg