<!-- TOC -->

- [1. 左值和右值](#1-左值和右值)
  - [1.1. 定义](#11-定义)
  - [1.2. 左值引用与右值引用](#12-左值引用与右值引用)
  - [1.3. 移动构造函数和移动赋值函数](#13-移动构造函数和移动赋值函数)
  - [1.4. 总结](#14-总结)
- [2. 智能指针](#2-智能指针)
  - [2.1. shared_ptr](#21-shared_ptr)
  - [2.2. unique_ptr](#22-unique_ptr)
  - [2.3. weak_ptr](#23-weak_ptr)

<!-- /TOC -->

# 1. 左值和右值
[参考](https://docs.microsoft.com/zh-cn/cpp/cpp/lvalues-and-rvalues-visual-cpp?view=vs-2019)
## 1.1. 定义
- glvalue:决定了*对象*、*位域*或者*函数*等标识的表达式
- prvalue:初始化*对象*、*位域*或者*计算操作符操作数的值*的表达式
- xvalue:glvalue/prvalue的子集，资源能复用的*对象*或者*位域*的表示符号
  - 存在地址，不再能被程序访问，但是可以用来初始化右值引用，能被表达式访问
  - 比如：返回右值引用的函数调用，数组的下标，成员和指向数组或对象为右值引用的成员
- lvalue:左值，是glvalue而非xvalue的那一部分子集
  - 程序能直接访问左值的地址
  - 比如：变量名，const变量，数组元素，返回左值类型的函数，位域，union类型和类成员
- rvalue:右值，是prvalue而非xvaluae的那一部分子集
  - 程序无法直接访问右值的地址
  - 比如：各类常量，非引用类型，在表达式计算期间创建但只能被编译器访问的临时对象

<div align="center">

![][LvalueAndRvalue]

</div>

[LvalueAndRvalue]: ./LvalueAndRvalue.jpg

## 1.2. 左值引用与右值引用
- 左值引用用法与指针类似
- 右值引用
  - 移动构造函数与移动赋值函数：
    - 参数为右值引用
    - 深拷贝
  - 解决Forwarding Problem:一个函数的参数使用引用T&，传递到调用的函数中使用const T&类型的引用，一般情况下需要用重载解决这种问题。但是如果采用右值引用的话，就不存在这个问题。
    - std::forward<T>():调用T的构造函数
  - 编译器把有名字的右值引用当成左值，把没有名字的右值引用当成右值。
  - 通过std::move可以把右值引用转左值，通过static_cast把左值转右值引用
  - 函数模板导出它们的模板参数类型，然后使用引用折叠规则

## 1.3. 移动构造函数和移动赋值函数
几种默认为delete的情况：
- 有拷贝构造/赋值函数且为定义移动构造/赋值函数
- 有类成员为定义自己的拷贝构造/赋值函数，且编译器不能为其合成移动构造/拷贝函数
- 被定义为delete或者不可访问
- 析构函数被定义为delete或者不可访问
- 类成员有const或者引用

## 1.4. 总结
左值与右值的划分更像是在不同层面代码上的划分：
- 左值：在高级语言代码这一层，能被程序直接通过变量名等获取地址的方式访问。可以理解为形式/标识的代表。
- 右值：在高级语言代码这一层无法直接获取其地址，一般用来初始化左值。在汇编语言代码这一层来看，通常是存在于寄存器中，无法通过堆栈地址直接被访问的临时变量/常量等。不过这样做的好处是，能在汇编这一层做更多的优化，从而提高效率，比如，配合只用移动拷贝函数/移动赋值函数等能省掉复制到临时变量（堆栈中）的这一阶段（因为本身就在堆栈中）。

# 2. 智能指针
使用动态内存的原因：
- 不知道对象的准确数量
- 不知道对象的准确类型
- 需要在多个对象间共享数据

## 2.1. shared_ptr
主要原理就是：对一个资源的指针进行引用计数。

可以跟new结合使用：
- 仅限于显示转换，不可采用隐式转换
  
> 需要注意：
>  - 同一块内存给多个shared_ptr进行初始化的情况，会导致内存块不可控的销毁
>  - 不要delete get()返回的指针
>  - 资源不是new分配的内存，需要安排一个删除器

## 2.2. unique_ptr
- 不能直接拷贝或者赋值，但是可以通过release()和reset()的方式来转移unique_ptr对指针的所有权。
  - release()方式释放的指针一定需要手动保存起来

## 2.3. weak_ptr
- 指向shared_ptr管理的对象，但是不会改变shared_ptr的引用计数。
- 因为无法确认指针是否存在，所以使用lock()返回指针