# 对象的构造和解构

## 全局变量
- C++程序中所有的global objects都被放置在程序的data segment中
- 其constructor和destructor都由编译器在main函数之前/之后隐式的调用

## 局部静态变量
- 所有的局部静态变量都会在程序起始时被初始化，现在已经被优化，变成执行期行为

## 对象数组
- 如果对象没有constructor和destructor，则跟内建的数据类型其实是一样的
- 在编译器中，使用一个类似如下的新函数，用来生成对象数组，并调用构造函数。析构的时候类似
> ```
> void* vec_new(void* array,
> size_t elem_size,
> int elem_count,
> void (*constructor)(void*),
> void (*desctructor)(void*, char))
> ```
 - 如果array不是具名数组的地址，则为0，这种数组将由应用程序的new 运算符被动态配置于heap中。

## Default Construtors和数组
如果class只提供了带参数的构造函数，上一节中的vec_new调用的constructor将会被改写成如下形式
```
class::class()
{
    class::class(arg, arg)
}
```

# new和delete运算符
- new的过程：
  1. 分配内存
  2. 给内存设置初值
- new和delete的底层实现还是malloc和free的那一套
## 针对数组的new
- 针对内置类型和不带constructor的struct，不会像上一节那样直接调用vec_new和默认的constructor。如果有constructor，则会调用vec_new
- 会调用new分配数组的内存空间
- vec_new 生成的空间会额外分配一个cookie用来存储当前分配的内存块大小

## 数组的destructor
- 为了保证destructor的正确，需要声明为virtual。不然按照vec_delete的方式，destructor拿的size大小是指针类型的，而不是实际的大小。

## placement new
- 返回指定内存的指针
- 在指定内存上调用constructor
- 最好用新的内存或者同类型object的内存，内存大小最好一致，不然会造成错误。比如用derived class来placement new一个base class的内存就会有问题，因为new 之后会调用constructor。