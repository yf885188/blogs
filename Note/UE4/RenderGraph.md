参考：UE4.sln中的RenderGraph.h

# RHI
RHI : Render Hardware Interface,针对不同平台提供相同的编程接口。

# Render Graph
Rendering Dependency Graph Framework。通过一种延迟执行的方式，来设置一些lambda域代码。这些代码，通常都被设计成pass，也主要用来执行RHI的GPU命令。

- FRDGBuilder::AddPass() : 添加新的pass
- FRDBuilder::AllcoParamters() : 添加pass的parameter，用来保证正确的声明周期，因为现在lambda代码已经被延迟执行了。
- FRDBuilder::CreateTexture()/FRDBuilder::CreateBuffer() : 只记录了资源描述符，只有当资源需要的时候，allocation操作才会被具体执行（懒汉？）。引用计数，资源可复用。
- pass中的Render Graph Resources资源必须是在pass的参数中明确声明过的，并且这些资源的声明周期必须跟lambda scope的生命周期是一致的。
- 不要给pass分配多余的资源，会造成性能浪费，通过ClearUnusedGraphResource的方式来自动清除无用的资源引用。没有被使用的资源也会发出警告，依靠FRDGResource::MarkResourceAsUsed()
- 在命令行中添加-rgdimmediate开启immediate模式，在Setup的时候进行debug。
- 传统方式的texture resource pool管理能够通过FRDGBuilder::RegisterExternalTexture()调用。
- 因为掌握了pass之间的依赖关系，具体执行的时候能按照不同的目标进行优化，比如：优化内存压力或者GPU并行执行。因此，不同目标下的pass具体执行顺序也不完全一样了。只能保证在intermediary resource下的执行跟immediate mode下GPU上的执行顺序一致。
- Render Graph Pass最好不要修改额外的数据，因为不同优化目标下，实际的执行顺序不一致。
- 如果一个Render Graph的资源生命周期比pass的时间要长，比如一些跨帧的资源，可以使用FRDGBuilder::QueueTextureExtraction()提取出来。
- 除了一些特殊情况，比如vr中的stereo rendering，不要在同一个pass中在不同资源上绑定多个work，这样的话会加深资源间的依赖关系，这些会导致一些其他GPU工作的overlap，也会导致一些局部资源的长时间占用，增加当前帧的内存压力


# 优点
- pass 之间的依赖关系
- 根据不同目标优化pass顺序
- pass 中设定了资源的依赖关系，更方便的进行资源的管理和优化

# 注意
- Render Graph Pass 跟其他pass的耦合尽量低，涉及到资源的管理和优化，不同目标下执行顺序的优化等等