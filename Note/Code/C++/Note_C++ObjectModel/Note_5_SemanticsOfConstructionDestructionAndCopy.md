# 纯虚拟函数的存在
- 可以定义和调用一个纯虚函数，但是只能以静态的方式进行，不能经由虚拟机制调用。

# 无继承情况下的对象构造
## 针对POD（Plain Of Data）
声明时会合成：
- trivial default constructor
- triaial desctructor
- trivial copy constructor
- trivial copy assignment operator

因为是POD,所以很多操作都被简化，比如trival copy contructor被简化为简单的位拷贝操作等等。

## 抽象数据类型（Abstract Data Type）
### Explicit initialization list
使用initializetion list会比一般的赋值操作要快，但同时也存在3个缺点：
- 只有当class members都是public时才奏效
- 只能指定常量
- 不是编译器自动施行的，因此初始化的失败可能性会比较高

## 继承的准备
- vptr，空间膨胀
- constructor都被附加了一些code，用来将vptr初始化，被加到所有base class constructor之后，所有derived class的explict code之前，同样使空间膨胀
- 合成一个copy constructor和一个copy assignment operator

# 继承体系下的对象构造
## 编译器对Constructor的扩充
按照顺序：
- virtual base class constructor被调用
- 根据申明顺序调用base class constructor
- 如果class object有vptr，就必须对vptr设置初值，指向适当的vtbl
- member initialization list中的data members初始化操作会被放到constructor的函数中，并以members的声明顺序为顺序
- 如果一个member并没有出现在member initialization list之中，但它有一个default constructor，那么就会调用default constructor

## 虚拟继承
- 虚拟继承会导致上面的扩充代码失效，因为virtual base class存在共享性的问题。
- 通过使用一个标志位来表示当前的derived class是否是最底层的class来判断，是否由当前的class来对共享的virtual class object进行初始化。

## vptr初始化语意
- 在一个class的constructor中，经由构造中的对象来调用virutal function，其函数实体应该是在此class中有作用的那个。
- 在调用操作限制在constructor中直接调用时，函数实体还没完全建构完成，这个时候进行调用，则是通过静态的方式进行调用，而无法用到虚拟机制

# 对象赋值语意
一个class对于默认的copy assignment operator，在以下情况不会出现bitwise copy语意：
- 当class内带一个member object，而其class有一个copy assignment operator时
- 当class的base class有一个copy assignment operator时
- 当class有vptr时，直接copy会导致两个vptr一样，但是可能是继承链中不同的继承层级
- 当class继承自一个virtual base class

# 解构语意学
并不需要保证跟constructor的对称关系，跟constructor中的扩展代码执行顺序正好相反。
- 如果class的member class objects有destructor，则会以声明顺序的相反顺序被调用
- 如果有任何直接的nonvirtual base class拥有destructor，则会以声明顺序的反顺序被调用
- 如果带有vptr，则重设vtbl。~~在程序的显式code之前被执行。(错误)~~
- 如果任何virtual base class拥有destructor，且当前的class是最尾端的class，那么他们会以其原来的构造顺序的相反顺序被调用

比较好的执行策略就是维护两份destructor实体：
- 一个complete object实体（针对最底层的class），总是设定好vptr，并调用virtual base class destructors
- 一个base class subobject实体，除非再destructor函数中调用一个virtual function，否则绝不会调用virtual base class destrcutors并设定vptr