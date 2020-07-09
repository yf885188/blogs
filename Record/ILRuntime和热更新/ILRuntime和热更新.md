# 热更新
通过补丁的形式更新相关资源或者修改bug等，不需要重新更新整个客户端，降低了项目更新的难度。

## 工作流
- 打包资源到远端
- 下载version.txt
- 对比本地跟远端的version.txt进行对比，获取要热更的资源
- 下载对应的资源

# ILRuntime
参考[官方文档](https://ourpalm.github.io/ILRuntime/public/v1/guide/index.html)。
ILRuntime是Unity用来解决代码热更新的一种方式。
> Unity代码跟资源进行分离，减少了asset中对代码的依赖程度。这样没有依赖的资源和代码，在打包热更的时候，只用更新ILRuntime的dll就行，不用担心依赖关心导致资源的更新。

## 工作原理
### IL托管栈

## 反射
实质就是ILRuntime和Unity主工程间夸Appdomain进行调用的一种解决方式。

辅助类：
ILRuntimeType、ILRuntimeMethodInfo、ILRuntimeFieldInfo等。

具体操作，参见[官方档案](https://ourpalm.github.io/ILRuntime/public/v1/guide/reflection.html).

> 跨Appdomian的实例化、委托和协程用Adapter

## CLR重定向
### 工作原理
- CLR重定向在Appdomain用一个重定向表来实现，CLRMethod在构造函数中就进行了重定向的绑定，CLRMethod的构造采用懒汉模式

### 工作流
- 使用ILRuntime的绑定接口生成绑定代码：代码中使用CLR重定向进行方法的重定向。
- 使用ILRuntime的绑定接口把CLR重定向的代码注册到Appdomain。
- Unity主工程获取ILRuntime的Appdomain，然后找到对应的type.method。
- 调用Appdomain.Invoke来调用对应的Method。调用过程中，通过IL解释器（ILInteperter）来获取Method正确的IL编码：如果进行了重定向，走重定向后的method；没有重定向，走CLRMethod.Invoke。

<div align="center">

![CLRBinding工作流][CLRBinding_WorkingFlow]

</div>

### 相比反射的优势
- 直接对ILRuntime的托管栈进行操作，避免了使用反射方式带来的GC消耗

<div align="center">

![CLRBinding][CLRBinding]
CLRBinding方式

![CLRMethodInvoke][CLRMethodInvoke]
反射方式

</div>

## 性能优化
### 减少GC
- 反射会使用List[] Objects来传参增加装箱拆箱消耗，使用CLR绑定可以绕过这个问题,CLR的底层实现就是采用的CLR重定向。
- 跨域的值类型也会导致大量GC，通过值类型绑定来解决这个问题。
- 使用InvocationContext:能减少GC的原因，也是直接对栈进行操作，相关参数入栈。

> 实质上是将在堆上进行内存分配的GC机制转移到了栈上。通过在栈上直接读取参数，分配的内存也是栈的内存，跟堆无关了。

## 工作流
- 加载程序集： 起相应的协程；加载热更dll；加载热更dll对应的pdb；加载程序集；
- 初始化ILRuntimeDLL: 委托适配器；协程适配器；值类型绑定；
- CLR绑定。

## 实际需要映射的类型
可以参考ILRuntime官方demo里面提供的Hotfix_Project.sln.
- CLR重定向
- CLR绑定
- 继承/实例化映射： 适配器模式
- 委托映射：适配器模式
- 协程映射： 适配器模式；协程的本质是个状态机；需要实现IEnumerator<System.Object>, IEnumerator, IDisposable等；结合之前，要实现的接口有Current、Dispose、MoveNext等。
- Jason映射
- MonoBehaviour映射：适配器模式；与协程映射类似，需要实现Monobehaviour的常用接口。
- 值类型映射: 值类型相关的操作CLR重定向。

# 实际流程
参考[Blog](https://www.icode9.com/content-4-694479.html)。不过最后热更新的时候，只用Build->Update a Previous Build，不用Tools->Check for Content Update Restrictions。

# 保留问题：
- 为啥ILRuntime访问Unity主工程只用Class.Method的方式，而反过来不是：Unity主工程走的是CLR或者MonoRuntime。
- 为啥ILRuntime要自己写个IL托管栈，而不用C#的Stack：基础类型会导致大量GC。
- 如果用C#的stack实现该怎么实现？

[CLRBinding_WorkingFlow]: ./ILRuntimeCLRBinding_WrokingFLow.jpg
[CLRBinding]: ./CLRBinding.jpg
[CLRMethodInvoke]: ./CLRMethodInvoke.jpg
