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
- 编译器还会合成内部使用的隐性data member，比如vtpr，要么在所有members最后，要么在所有members最前，视编译器决定
- 对于access sections：C++Standard允许sections中的变量自由排列，但是编译器一般都是把access section进行合并

# Data Member的存取
## Static Data Members
- 存放在程序**堆**的data segment中
- 在这种情况下，**通过指针和通过对象来存取member，结论完全一致**

## Nonstatic Data Members
- 在member function中直接处理nonstatic data member，都可以视为是this->data_member的形式
- nonstatic data member的地址通常是class object的地址加上偏移量计算得到。偏移量在编译时期就已经计算好了。
- 在这种情况下，**通过指针和通过对象来存取member，结论会发生很大变化**。主要是当member是从virtual base class继承而来时的这种情况会导致这种巨大的差异，因为在这种情况下，只有在运行期才能准确的知道，member对应具体是哪个class，从而知道member真正的offset值。通过对象的方式，其类型已经确定，在编译器就已经能知道offset的具体值。
  