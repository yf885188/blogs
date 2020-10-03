<!-- TOC -->

- [1. 左值和右值](#1-左值和右值)
  - [1.1. 定义](#11-定义)
  - [1.2. 左值引用与右值引用](#12-左值引用与右值引用)
  - [1.3. 移动构造函数和移动赋值函数](#13-移动构造函数和移动赋值函数)
  - [1.4. 引用规则](#14-引用规则)
  - [1.5. 总结](#15-总结)
- [2. 智能指针](#2-智能指针)
  - [2.1. shared_ptr](#21-shared_ptr)
  - [2.2. unique_ptr](#22-unique_ptr)
  - [2.3. weak_ptr](#23-weak_ptr)
- [3. 三/五/零法则](#3-三五零法则)
  - [3.1. 三法则](#31-三法则)
  - [3.2. 五法则](#32-五法则)
  - [3.3. 零原则](#33-零原则)
- [4. 虚函数表](#4-虚函数表)
- [5. 强制类型转换](#5-强制类型转换)
  - [5.1. static_cast](#51-static_cast)
  - [5.2. const_cast](#52-const_cast)
  - [5.3. reinterpret_cast](#53-reinterpret_cast)
  - [5.4. dynamic_cast](#54-dynamic_cast)

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
    - std::forward<T>():调用T的构造函数。如果自变量是右值或者右值引用，则有条件的将自变量转换为右值引用。
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

## 1.4. 引用规则
[规则](https://www.jianshu.com/p/b90d1091a4ff)：
- 引用折叠：引用的引用会被折叠。除了一种特殊情况，其他所有引用的引用会被折叠为一个普通的左值引用类型。
  - T& &, T& &&, T&& &都被折叠成T&
  - T&& &&被折叠成T&&
- 右值引用的特殊类型推断规则：当将一个左值传递给一个参数是右值引用的函数，且此右值引用只想模板类型参数时，编译器推断模板参数类型为实参的左值引用
- 虽然不能隐式的将一个左值转换为右值引用，但是可以通过static_cast/std::move将左值转换为右值引用。

## 1.5. 总结
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

# 3. 三/五/零法则
三种可以控制类的拷贝操作：拷贝构造函数、拷贝赋值运算符和析构函数。此外，新标准下还有移动构造函数和移动赋值运算符。

## 3.1. 三法则
拷贝构造函数、拷贝赋值运算符和析构函数基本上是同时需要。

## 3.2. 五法则
拷贝构造函数、拷贝赋值运算符和析构函数会阻止移动构造函数和移动赋值函数的隐式定义，任何想要移动语义的类必声明全部五个特殊成员函数。

## 3.3. 零原则
类中五种操作的表现应该与其成员的对应五种操作保持一致。

> 此外，还要注意：
> - 需要析构函数的类也需要拷贝和赋值操作（三原则）
> - 需要拷贝操作的类也需要赋值操作，反之亦然（三原则）
> - 析构函数不能是删除的成员（三原则）
> - 合成(默认)的拷贝控制函数可能是删除的：如果一个类有成员是不能默认构造、拷贝、复制或者销毁，则对应的成员函数将被定义为删除的。（零原则）
> - 如果一个类有const成员，则不能使用合成(默认)的拷贝赋值函数。（零原则）

# 4. 虚函数表
[参考](https://blog.csdn.net/primeprime/article/details/80776625)

- 有虚函数的类包含虚表
- 虚表的赋值发生在编译器的编译阶段
- 虚表的指针是类共享，而非具体对象
- 利用虚表和虚表指针来实现动态绑定

# 5. 强制类型转换
static_cast、dynamic_cast、const_cast和reinterpret_cast。

## 5.1. static_cast
- 大的算术类型转换给小的算术类型
- 编译器无法自动执行的类型转换
- 不保证安全
  
## 5.2. const_cast
常量和非常量的转换。不保证安全。

## 5.3. reinterpret_cast
在位的层次上进行转换，不保证安全。

> reinterpret_cast 和 static_cast的区别
> reinterpret_cast在位上进行操作，保证转换前后的值相同；
> static_cast不保证值相同

## 5.4. dynamic_cast
主要针对基类和派生类型进行变换。

三种使用形式
- dynamic_cast<type*>(e) : e必须是指针
- dynamic_cast<type&>(e) : e必须是左值
- dynamic_cast<type&&>(e) : e不能是左值

必须满足以下三种条件之一：
- type的公有派生类
- type的公有基类
- type类

否则，一定不成功。

转换也有失败的情况，如果是指针则返回0。

> 除了dynamic_cast，还可以使用typeid进行RTTI的类型识别判断。