# 分类
- function adapter
- container adapter
- iterator adapter

# 使用functor adapter而不使用func ptr
- functor adapter最终都是返回的一个实例，记录了通过template推导的参数型别等信息。而func ptr实现类似的效果会更加麻烦
- functor adapter因此在配接上更加的灵活

# mem_fun可以把成员函数当做仿函数使用
- 成员函数是基类继承/重写之后