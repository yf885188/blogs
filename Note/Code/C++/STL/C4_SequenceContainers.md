# 分类
![][KindsOfContainer]

[KindsOfContainer]: ./KindsOfContainer.jpg

# vector
- Random Access Iterator
- 动态增加大小，并不是在原空间后面直接增加空间
  - 以原空间大小的两倍申请空间
  - 把原空间内容给拷贝过来
  - 在原空间之后构造新元素
  - 释放原空间

此外：
- 容量和size的区别
- 插入等操作可能会造成vector增容，导致迭代器失效。

# list
- Bidirectional Iterator
- 不存在迭代器因为增容而失效的问题
- SGI ListShift一个环状双向链表

# deque
- Random Access Iterator
- 由一段一段的定量连续空间构成,制造一种整体连续的表象，然后通过对operator++和operator--的封装来维持这种表现。

![][DequeMemStructure]

[DequeMemStructure]: ./DequeMemStructure.jpg

## 扩容操作
- 定义两个空间配置器：
```
typedef simple_alloc<value_type, Alloc> data_allocator
typedef simple_alloc<pointer, Alloc> map_allocator
```
- ctor中只有一个fill_initialize，它包含
  - create_map_and_nodes: 创建map结构；创建缓冲区节点；设置start，finish节点相关数据。
  - 为每个节点设立初值
- 通过push_back_aux/push_front_aux来扩容
- 通过reserve_map_at_back()/reserve_map_map_at_front()对map进行重新整治

# stack
以deque作为底层结构，因此，一般也不作为容器，而是作为container adapter。

> 只能从顶端获取元素，因此没有迭代器。

# queue
以deque作为底层结构，因此，一般也不作为容器，而是作为container adapter。

> 只能从前端和后端获取元素，因此没有迭代器。

# heap
不是STL容器组件，作为priority queue的辅助结构。

## 隐式表述法
用array/vector来表示完全二叉树。

## 实现
vector + heap算法（插入/删除/取值）

## heap算法
- push_heap:上溯过程
- pop_heap:下溯过程，拿出vector尾端的添到第一个，然后下溯下去。
- sort_heap：持续的pop_heap便获得一个有序的heap
- make_heap：用隐式表述法来完成

> 所有元素必须遵循特别的排列规则，所以heap不提供遍历功能，也不提供迭代器。

# priority_queue
以一个max-heap实现。因此也是一个container adapter，没有迭代器。

# slist
单向链表。
- 迭代器属于Forward Iterator
- end()实际上是重新构造了一个node
