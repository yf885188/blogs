# Member func的几种调用形式
- Nostatic Member Function
- Static Member Function
- Virtual Function
## Nonstatic Member Function
member function要保证跟nonmember function一样的效率，并且执行的时候，一般被内化为nonmember function的形式。转换步骤：
- 改写函数原型以安插一个额外的参数到member function中，这个额外的参数通常被称为this指针。
- 将每一个对nonstatic data member的存取操作改为经由this指针来存取。
- 将member function重新写成一个外部函数。
## Virtual Member Function
调用过程会加一层间接性，在vptr中索引func的地址然后进行后续的调用。
## Static Member Function
- 对象调用和指针调用会被转换成一样的调用方式。
- this指针的作用是把member function中存取的nonstatic class member绑定于object内对应的members之上
- 其指针不是一个指向class member function的指针，而是一个指向nonmember function的指针。类似全局的一个指针地址。

主要特性：没有this指针。基于主要特性的其他特性：
- 不能直接存取class中的nonstatic members
- 不能被声明为const、volatile或virtual
- 不需要经由class object才被调用

# Virtual Member Function
- virtual member function决定了当前类是否需要多态，因此也决定了当前类是否需要创建为多态服务的额外结构信息
- 额外的结构信息包括：指针所指的对象真实类型；指针所调用函数的实体位置。因此可以在每个多态的class object上增加两个members：一个表示class类型的字符串或数字；指向表格的指针，表格中带有程序的virtual functions的执行期地址。virtual function的地址在编译器就已经确定下来了。为了找到这些函数地址在每个class object中安插了一个由编译器内部产生的指向该表格的指针；每个virtual function被指派一个表格索引值。以上这些操作都是编译器执行，不需要额外的介入。
  - virtual function包括：
    - 当前class定义的函数实体，可能回重写base class的virtual function
    - 继承自base class的函数实体
    - pure_virutal_called()函数实体：pure virtual function的空间保卫者角色（pure virtual function没有实体，就用pure_virtual_called()来占位）；或者做执行期异常处理函数。
> 可以思考一下C#中进行函数调用的实现机制：每个class object保存一个指向类型对象的指针，类型对象记录了类型对应的member的地址等等

单一继承中，vtbl可能出现的情况：
- 直接继承virtual function的函数实体，也即把base class vtbl中的函数地址拷贝自己vtbl中对应的slot上
- 重写base class的实体，也即用自己的函数地址填写base class vtbl中已经存在且不是pure_virtual_called()的slot
- 加入新的virtual function的地址。在vtbl中新增slot，在新增slot中填入新的函数实体地址。

## 多重继承下的virtual functions
主要问题集中在以下几个方向：
- virtual destructor
- 后续base class中的非同名函数
- base class之间存在的同名函数

### 解决原理：base class 和 derived class 指针转换的内部逻辑
单一继承情况下，可以直接从开头的地址截取subobject就行。然而，在多重继承的情况下，存在多个base class subobject，所以在转换的时候，需要确定指针的偏移值,用于调整this指针，而这些都使多重继承相对于单一继承多了很多负担。

#### 负担1：引入Thunk技术，带来了其他开销
Thunk 技术:就是一小段assembly码，主要作用：1.以适当的offset调整this指针；2.调到virtual function。这些都对调整this指针产生了一些负担。

#### 负担2：同一个函数在virtual table中需要维护多个对应的slot
在多重继承之下，一个derived class内含n-1个额外的virtual table（第一个base class不需要额外的vtbl）。

<div align="center">

![][MI_Vtbls]

</div>

[MI_Vtbls]: ./MI_Vtbls.jpg

## 虚拟继承下的virtual functions
太复杂，给出建议：不要在virtual base class中带入data member。