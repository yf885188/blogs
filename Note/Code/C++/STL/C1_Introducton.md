# STL的六大组件
- 容器(containers):class template
- 算法(algorithms):fuction template
- 迭代器(iterators):将operator*, operator->, operator++, operator--等指针相关操作予以重载的class template
- 仿函数(functors):重载了operator()的class或者class template
- 配接器(adapters)：用来修饰容器或仿函数或迭代器接口的东西
- 配置器(allocators)：负责空间配置与管理

<div align="center">

![][STL_Components]

</div>

[STL_Components]: ./STLComponents.jpg

# 文件分类
- C++标准规范下的C头文件
- C++标准程序库中不属于STL范畴者
- STL 标准头文件
- C++ Standard定案前，HP规范的STL头文件
- SGI STL内部文件