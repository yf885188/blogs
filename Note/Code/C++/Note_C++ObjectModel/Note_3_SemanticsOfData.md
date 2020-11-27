- 空类至少有1B
- 虚继承的情况：
  - 会有指针消耗：virtual base class subobject的地址或者base class的偏移量
  - 会在指针之后添加空类的1B，但是视编译器而定，有的会添加，有的不会
  - 有时候alignment会把虚继承的5B给对齐到8B
- 针对菱形虚继承的情况，一个virtual base class subobject只会在derived class中存在一份实体

# Data Member的绑定
- member function的分析会在class声明完成之后才开始分析，但是member function的argument list会在初次遇到的时候就开始绑定。在早期的编译器中，这就会导致一个问题，就是同名的全局变量和类内变量在argument list绑定的时候率先绑定为全局的那个。因此也产生了两种防御编程方式：
  - 先声明member data，放在类声明的起始处
  - 把inline 函数的生命放在类声明完成之后

# Data member的布局
- 类的static member data不会放到对象布局之中，而是放到程序**堆**的data segment中
- member data的存储不一定是连续的，只要保证**较晚出现的members在class object中有较高的地址**就行，在其之间可能会出现边界调整填补的一些bytes，视编译器决定
- 编译器还会合成内部使用的隐性data member，比如vptr，要么在所有members最后，要么在所有members最前，视编译器决定
- 对于access sections：C++Standard允许sections中的变量自由排列，但是编译器一般都是把access section进行合并

# Data Member的存取
## Static Data Members
- 存放在程序**堆**的data segment中
- 在这种情况下，**通过指针和通过对象来存取member，结论完全一致**

## Nonstatic Data Members
- 在member function中直接处理nonstatic data member，都可以视为是this->data_member的形式
- nonstatic data member的地址通常是class object的地址加上偏移量计算得到。偏移量在编译时期就已经计算好了。
- 在这种情况下，**通过指针和通过对象来存取member，结论会发生很大变化**。主要是当member是从virtual base class继承而来时的这种情况会导致这种巨大的差异，因为在这种情况下，只有在运行期才能准确的知道，member对应具体是哪个class，从而知道member真正的offset值。通过对象的方式，其类型已经确定，在编译器就已经能知道offset的具体值。

# “继承”与Data member
- 派生类一般都是基类和自己members的总和，这两者的排序在C++ standard中并未强制指定，视编辑器而定，一般都是先base class的members。但是virtual base class的情况除外。

总共分4种情况可以讨论：
- 单一继承且不含virtual functions
- 单一继承并含virtual functions
- 多重继承
- 虚拟继承

## Inheritance without Polymorphism:只要继承不要多态
也即，单一继承不含virtual functions
- 内存布局就是member data的顺序布局
- 但是要注意这种情况，类的大小会自动对齐到word大小，产生很多空间浪费。但是不对齐又会产生很多其他bug，比如父类子类指针的传值带来的数据覆盖问题。除此之外，因为实际上定义了新类，所以也会产生一些操作成本。

## 单一继承并含多态
额外负担：
- vitual table存放virtual function
- vptr
- constrtuctor中存在给vptr设定初值的隐性code
- destructor清除vptr

## 多重继承
父类与子类的指针转换要视内存布局而定：
- 起始地址一样，直接拷贝
- 起始地址不一样，要计算偏移值

## 虚拟继承
分为两个部分：
- 不变局部：从object开始有固定的offset
- 共享局部：virtual base class subobject部分。该部分的数据会因为每次的派生操作而有变化，所以只能被间接存取。有三种主流策略：
  - 原始思路：先安排好derived class的不变部分，然后建立共享部分。共享部分可以通过在derived class object中安插一些指向virtual base class的指针来进行。主要有两个缺点：每个virtual base class都有一个指针；虚拟链长了之后，间接存取调用链也会增加
  - 第一种方案：针对原始方案的第二个缺点，计算调用链，并放到derived class object中，以空间换时间
  - 第二种方案：在第一种方案上更进一步，使用virtual base class table存储每个base class的指针，并在class object中安插一个指向virtual base class table的指针
  - 第三种方案：在virtual function中放置virtual base class的offset而不是地址

# 指向data member的指针
- 区分没有指向data member的指针和指向第一个data member的指针：针对后者offset+1
- &Class::DataMember表示一个member的offset，而&ClassObject.DataMember表示的是内存地址。前者比后者多一层间接性，效率更低