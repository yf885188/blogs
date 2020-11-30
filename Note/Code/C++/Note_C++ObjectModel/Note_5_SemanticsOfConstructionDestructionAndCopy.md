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