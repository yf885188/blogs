# Default Constructor的建构操作
- class object 的class member会在default constructor中被默认调用，但是如果定义了user-defined constructor则不会
- member objects的初始化会根据在class中的声明次序来调用各个constructors

## 编译器默认合成Default Constructor的四种情况
- member object种存在default constructor
- base class存在default constructor
- 有virtual function：生成vtbl和vptr
- 继承链中存在virtual base class

这几种情况都会默认扩展其他的constructor，按照隐形的default constructor建构之后，进行原有的constructor。也即在显式的constructor之前插入这些隐式的constructor操作。

# Copy Constructor的建构操作
- memberwise copy：逐成员的拷贝，会涉及到member object的copy操作
- bitwise copy： 逐位的拷贝，效率会更高

## 不用bitwise copy的四种情况
跟合成Default Constructor的情况类似
- member object中存在copy constructor
- 继承的base class中存在 copy constructor
- 有virtual function：生成vtbl和vptr
- 继承链中存在virtual base class

# 程序转化语义
- 明确的初始化操作：初始化时，赋值构造和拷贝构造函数进行的操作
- 参数初始化：
  - 非引用方式的传参会做多个步骤：生成暂时对象，暂时对象的copy constructor，然后以bitwise copy的方式传给形参
  - 引用的方式：直接省掉了这些步骤。直接以copy construct的方式把实参构建到指定位置（存疑：vs2019实践中，函数外跟函数内引用的位置地址一样，也没有进入copy constructor,跟编译器有关？）
- 返回值的初始化：也默认使用到了copy construct。
```
# 初始代码
X bar()
{
    X xx;
    return xx;
}

# 伪代码
void bar(X& _res_)
{
    X xx;
    xx.X::X();
    _res_.X::X(xx);
    return;
}

```
  - 优化：
    - 在使用者层面
    - 在编译器层面：NRV（Named Return Value）优化。优化copy constructor的实现过程，比如用内存拷贝的方式代替memberwise的方式等等。但是这种方式并不准确，主要原因是编译器的优化完全是黑盒。因此也会导致使用问题，比如代码层面的优化会作废等等。
```
# 初始代码
X bar(const T& y, const T& z)
{
    X xx;
    return xx;
}

# 在使用者层面进行优化
# 伪代码
void bar(X& _res_, const T& y, const T& z)
{
    _res_.X::X(y, z); //跳过了默认构造函数和拷贝构造函数
    return ;
}

# 在编译器层面进行优化
# 伪代码
void bar(X& _res_)
{
    _res_.X::X();
    //后续直接对_res_进行处理，而不用拷贝构造的过程
}
``` 

## 是否编写Copy Constructor
- 存在大量的返回值的情况，有NRV优化
- 注意是有隐式的member产生比如vtpr等等

