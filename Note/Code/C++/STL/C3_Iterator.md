# 设计思维
STL的中心思想：将数据容器和算法分开，彼此独立设计，最后再以胶水撮合在一起。

- 数据容器的泛型化：class templates
- 算法的泛型化：function templates
- 胶水：迭代器

# 迭代器的本质是smart pointer
每种STL容器都有一个专属的迭代器。

> 因此需要重载 operator * 和 operator ->

# Traits编程技法
- 迭代器value type：迭代器所指对象的型别
  - 通过function template的参数推导机制可以比较快速的确定value type
  - 使用声明内嵌的方式来确定返回值的value type
  - 针对原生类型的指针，则使用偏特化来处理

![][IteratorTraits]

[IteratorTraits]: ./IteratorTraits.jpg

## 5类traits type
- iterator_category:采用继承机制的tag class
- value_type:
- difference_type:
- pointer:
- reference:

> - 设置traits typs是iterator的责任
> - 设置iterator则是容器的责任

# __type_traits
- traits 负责萃取迭代器的特性
- __type_traits负责萃取type的特性
  - 是否具备有效的default ctor/copy ctor/assignment operator/dtor

## 基本定义
- __type_traits<T>::has_trivial_default_constructor
- __type_traits<T>::has_trivial_copy_constructor
- __type_traits<T>::has_trivial_assignment_constructor
- __type_traits<T>::has_trivial_destructor
- __type_traits<T>::is_POD_type

> 根据这些特性，在做数据处理的时候，判断是进行最有效率的方式，还是最安全的方式。