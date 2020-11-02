# Object Model
- 两种class data member
  - static
  - non-static
- member function
  - static
  - non-static
  - virtual

# 不同的Object Model比较
- Simple Object Model
  - 每个member以slot的形式存储在object中，但是只会存储对应member的指针
  - 没有实际运用
- Table-driven Object Model
  - data member table ： data的具体值
  - member functio table ： 记录函数地址
  - 也没有实际运用
- C++ Object Model 

# C++ Object Model
从Simple Object Model派生而来，对内存空间和存取空间做了简化
  - non-static data member置于每个class object中；static data member存放在所有的class object之外；static 和 non-static function members被放于所有的class object之外。
  - virtual functions : 每个class产生一个指向virtual table(vtbl)的指针(vptr)，virtual table中用于存放指向virtual functions的指针。vptr的设置和重置由class的constructor、destructor和copy assignment来自动完成。每个class关联一个type_info object，用于支持RTTI，放于vtbl的第一个slot。

<div align="center">

![][ObjectModel]

</div>

[ObjectModel]: ./ObjectModel.png

## 继承
- 单一继承
- 多重继承
- 虚拟继承：多种方式
  - 每个base class的指针在derived class中占用一个slot
  - 为derived class维护一个base class table(bptr),bptr中的每个slot对应一个base class
  - 早起的C++继承模型把base class member直接放到derived class object中。

## 三种程序设计范例
- Procedural Model : 程序模型
- Abstract Data Type Model(ADT) ： 抽象数据类型模型
- Object-oriented Model ： 面向对象模型

## 多态支持
- 隐式转换
- virtual function
- dynamic_cast 和typeid

> class object的内存组成
> - nonstatic data members的总和
> - alignment填补的padding
> - 支持virtual产生的overhead

> 指针和引用更多表示的是对当前内存的“大小和内容解释方式”。

