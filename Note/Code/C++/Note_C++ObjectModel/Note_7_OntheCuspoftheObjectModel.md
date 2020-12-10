# 扩充特性
- template
- exception handling(EH)
- runtime type indetification(RTTI)

# template
主要讨论方向：
- template的声明
- 如何具现出(instantiates) class object以及inline nonmember，以及member template functions
- 如何具现出 nonmember 、member template functions和static template class members

# 具现行为（Template Instantiation）
具现行为：根据template类型，产生出具体的函数/类实体。
- template<class> *ptr的方式不会产生具现行为。但是引用的方式会产生具现行为。
- template class中没有使用到的member function不会被具现，只有使用到时才会被具现。
  - 效率考量
  - 针对尚未实现的技能有更高的鲁棒性。

具现的时机，两种策略：
- 在编译的时候
- 在链接的时候

## Template的错误报告
- 所有与类型有关的检验，如果涉及到template参数，都必须延迟到真正的具现操作发生，才得为止
- nonmember 和 member template在具现发生之前也没有做完全的类型检验

## Template的名称决议方式
template体现的2个作用范围：
- scope of the template definition：定义域
- scope of the template instantiation：具现域
决议细节：
- template之中，对于一个nonmember name的决议结果是根据这个name的使用是否与具现出template的具体参数类型有关来判断的：无关则使用定义域来决定name，相关则使用具现域来决定name。
- 决议只跟函数签名有关（signature），与返回值类型无关。

## 具现行为
三个主要问题
- 编译器如何找出函数的定义：包含template program text file，类似头文件；或者根据命名规则来保证
- 编译器如何能够只具现出程序中用到的member function
  - 简单粗暴，直接把已经具现出来的class的所有member functions都具现出来
  - 仿真链接操作，把函数真正需要的函数给生产出来
- 阻止member definition在多个目标文件中具现出来
  - 产生多个实体，链接的时候只保留一个
  - 在链接的时候生成，并通过策略确定哪个实体是真正的需求并保留

> 比较了auto template的几种方案之后得出的结论：还是手动具现化靠谱。

# 异常处理(Exception Handling)
编译器为支持异常设置的机制：
- 找出catch子句，用于处理被丢出来的exception。
- 查询exception objects的方法，知道其实际类型
- 管理被丢出的exception ojbect，包括他们的生产、存储、可能的解构、清理和一般的存取。

## C++ EH的语汇组件构成
- throw子句
- 一个或者多个catch子句
- 一个try子句

这样的机制在一定程度上改变了程序的运行语意、资源管理

## EH的流程
- 检验发生throw的函数：一个exception被丢出的时候，exception object会被产生出来并放在相同形式的exception数据堆栈中。
- 决定throw操作是否发生在try区段：program counter对函数进行区分，以决定thore是否发生在try区段
  - 若是，编译器必须把exception拿来和每一个catch子句比较：每一个函数会产生一个exception表格用检测exception object与catch子句的对应关系
    - 如果比较吻合，流程控制应该交到catch子句中
  - 如果throw的发生并不在try区段中，或没有一个catch子句吻合，那么系统必须
    - 摧毁掉所有的active local objects
    - 从堆栈中将当前的函数unwind掉
    - 进行到程序堆栈中的下个函数中去，然后重复上面的检查决定throw操作是否发生在try区段的操作

# RTTI
## Type-Safe Downcast
为实现这套机制增加了额外的开销：
- 用来存储类型信息的额外空间
- 需要额外时间来决定执行期的类型

> 可以通过在vtbl中添加class object的指针来记录类型信息。

## Type-Safe Dynamic Cast
判断以下一个转型操作是static还是dynamic，必须视指针是否指向一个多态class object而定。
```
pfct pf = pfct(pt);
```
> 相对于static cast，dynamic cast能保证运行期的安全性，会做安全检查。

### reference不是pointer
- 转换失败的时候，reference不能像pointer那样返回0表示失败，只能throw bad_cast exception

### TypeId运算符
使用TypeId进行类型判断，能绕过reference必须返回0或者throw bad_cast exception的问题。

## Type Info结构
- 内建类型也有Type Info结构，不过不是在运行期生成的，在程序执行前就已经静态生成好了。