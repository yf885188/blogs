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
