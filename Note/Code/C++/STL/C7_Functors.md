# 为啥不用函数指针而用仿函数
- 函数指针无法满足STL对抽象性的需求
- 不能与STL其他组件搭配

# 规则
- 重载function call 运算符operator()

# 可配接的关键
stl中定义了两个class，分别代表医院仿函数和二元仿函数，不支持三元仿函数。没有任何data members或者member functions，只有一些型别定义。

- unary_function
- binary_function

# 证同元素 identity element
数值若与该元素做op运算，会得到数值自己。

# 证同（identity） 选择（select） 投射（project）