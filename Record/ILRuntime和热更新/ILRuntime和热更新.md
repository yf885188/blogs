# 热更新
通过补丁的形式更新相关资源或者修改bug等，不需要重新更新整个客户端，降低了项目更新的难度。

## 工作流
- 打包资源到远端
- 下载version.txt
- 对比本地跟远端的version.txt进行对比，获取要热更的资源
- 下载对应的资源

# ILRuntime
ILRuntime是Unity用来解决代码热更新的一种方式。
> Unity代码跟资源进行分离，减少了asset中对代码的依赖程度。这样没有依赖的资源和代码，在打包热更的时候，只用更新ILRuntime的dll就行，不用担心依赖关心导致资源的更新。

## 反射
实质就是ILRuntime和Unity主工程间夸Appdomain进行调用的一种解决方式。

辅助类：
ILRuntimeType、ILRuntimeMethodInfo、ILRuntimeFieldInfo等。

具体操作，参见[官方档案](https://ourpalm.github.io/ILRuntime/public/v1/guide/reflection.html).

## 性能优化
### 减少GC
- 反射会使用List[] Objects来传参增加装箱拆箱消耗，使用CLR绑定可以绕过这个问题。
- 跨域的值类型也会导致大量GC，通过值类型绑定来解决这个问题。
- 使用InvocationContext
