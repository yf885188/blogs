# 框架
![][frame]

[frame]: ./images/frame.jpg

# 设计模式 - Proactor前摄器设计模式

![][ProactorDesign]

[ProactorDesign]: ./images/ProactorDesign.jpg

与reactor的区别：
- 主动去查询事件完成没有，而reactor是主动查询事件就绪没有
- 任务分发的并行性：reactor使用同步I/O多路分发，Proactor实现异步I/O分发
- reactor是被动的，proactor是主动的

# 线程
- 线程安全：使用io_context保证对同一object的并行访问安全
- 线城池
- 内部线程
  - 从单线程中调用io_context::run()来避免同步带来的开发复杂度问题
  - 线程启动之后应该立即进行初始化
  - boost::asio将线程的创建和管理进行了解耦

# strands
- 定义： 一系列的event handler的invoke，而不是并行的invocation
- 好处：可以不使用显式锁

## 使用方式
可以有显式调用和隐式调用方式
- 隐式方式：
  - 使用io_context::run()进行调用，io_context保证handlers只能从run()中被invoke
  - 类似Http这种链式进行的异步操作
- 显式方式：使用strand<>或者io_context::strand

注意事项：
- 在一些组合式的异步操作中，如果一个completion handler通过了一个strand，那种中间的handlers也应该走过同样的strand，这样也是为了保证线程安全。为了达到这种效果，所有的异步操作可以通过get_associated_executor函数获取handler的相关executor。

# Buffers
- 定义: I/O操作中数据传输的内存
- 优化：为了允许高效的网络应用开发，asio包含了对scatter-gather操作的支持，这些操作包含了一个或者多个buffer
  - scatter-read ：采集数据到多个buffer
  - gather-write ：传输多个buffer

## 分类
为了对buffers进行上层的抽象，并方便上层的使用，把buffer进行了分类：
- mutable_buffer
- const_buffer

> - mutable_buffer可以转换为const_buffer，反之不行
> - 对于buffer溢出有保护机制：
>   - 针对一个buffer实例，只能创建另一个大小不超过的buffer。
>   - 可以自动调整从**POD元素的array**/**string**创建而来的buffer
> - 通过data()函数来访问底层的内存

## 使用
- streambuffer：对input和output进行封装
- buffers_iterator<>：对buffer进行迭代遍历
- buffer debugging: VS8.0之后支持，或者在GCC中启用了_GLIBCXX_DEBUG宏


# stackless coroutines
用同步的方式来实现异步逻辑。

> 与C#中的协程进行类比，底层实现是个状态机

关键字
- reenter
- yield
- fork

# stackful coroutines
使用spawn()函数来运行

## spawn函数
- 第一个参数：可以是strand，io_context或者completion handler。指定了协程保证执行的context。
- 第二个参数：有boost::asio::yield_context作为参数的函数

## stackless coroutines和stackful coroutines的区别
- 手动和自动的区别？

# Coroutines TS Support
TS ： type script

新的数据结构：
- awaitable class template
- use_awaitable completion token
- co_spawn function