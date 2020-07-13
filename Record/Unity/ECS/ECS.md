# DOT
Data-Oriented Tech Stack。一种面向数据/属性的编程模式。

# ECS
## 组成
- Entities ： 游戏的组成,数据/属性的集合体
- Components : 管理Entities的相关数据
- Systems : 逻辑管理

## Entity
### Archetype
实体上各种Components的组合又称[Archetype(原型)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityArchetype.html)。EntityManager通过Archetype将具有相同Components集/Archetype的实体组织起来。

### 创建Entity的方式
单个创建：
- 用ComponentType创建
- 用EntityArchetype创建
- 用Instantiate创建
- 先创建不带component的实体，然后添加component
批量创建：
- CreateEntity + NativeArray
- Instantiate + NativeArray
- CreateChunk

### 改变Entity Components
添加或者移除Entity的components时，会导致结构的变化，改变SharedComponentData：
- 改变Entity的archetype
- 给新的archetype分配新的连续内存
- 压缩原有的内存

> 有时也会导致Entity的销毁操作，为了保证迭代正常，这些操作会放到EntityCommandBuffer中存储，然后等上一个迭代完成之后进行处理。

### EntityQuery
ECS的数据都存在Components中，ECS在内存中通过Entity的原型来组织这些数据。通过EntityQuery能方便的过滤并获取这些数据。并且这些数据是保证能并行的，这里的并行指的是不同在哪个array中，相同的索引都是对应相同的Entity。

#### 应用：
- 开一个job来处理这些Entitis和Components
- 获取一个包含了所有的已选择Entities的NativeArray
- 通过ComponentType获取一个已选Components的NativeArray

#### 使用方式
具体参看[官方文档](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_entity_query.html)。

- GetEntityQuery ： 在System类里使用。会缓存具有相同Filter设定的Query。
- EntityQueryDesc
- QueryOptions : IncludePrefab;IncludeDisable;FilterWriteGroup.
- CombiningQueries : OR
- CreateEntityQuery : 不在System类里使用。
- Filters : 可以设置Filters相关属性的值。SharedComponentFilter; ChangeFilter（只能根据system是否有对相关component进行写操作，但是写操作进行之后数据是否改变就不一定了。因此，尽量吧component设置成只读的，能加速这种判断的效率。）。
- Executing Query

##### Executeing Query
- Job :
- Method : ToEntityArray();ToComponentDataArray<T>();CreateArchetypeChunkArray();

## Components
Components用来组织游戏或者App的数据。Entity作为components集合的索引，Systems提供行为逻辑。
ECS中的Components是一个结构，包含以下标记接口之一：
- IComponentData : 针对GeneralPurposeComponents和ChunkComponents
- IBufferElementData : 针对DynamicBuffers
- ISharedComponentData : 通过Archetype内的值来对Entities进行组织和分类
- ISystemStateComponentData ：把系统特定状态和实体进行绑定，并检测单个实体的创建和销毁状态
- ISharedSystemStateComponentData ： 共享数据和系统数据的结合
- BlobAssets

EntityManager通过archetype来组织entities到一块块连续的Chunks（内存块），如图所示：

<div align="center">

![ECS 内存管理模型][ECSMemoryManagement]

</div>

此外：
- SharedComponents和ChunkComponents不在图中，是因为ECS不在chunk中存储这两类Conponents。
- DynamicBuffers也可以选择存在Chunk外。
- 及时上面说的几类Components不在Chunk内，但是使用EntityQuery的时候也可以认为这几类跟其他的Components能统一进行处理。

### IComponentData 
- 为Entity保存Instance Data的结构。
- 不应该存在除了公用方法以外接近Data的方式。
- 在面向对象的Unity系统中，这类似于一个只包含变量的Component类。
- 实质是struct而不是class，不要包含引用，要注意。原因是，ComponentData处在ChunkMemory中，而ChunkMemory不支持GC。

#### Managed IComponentData
- 声明为class
- 能以一种零碎的方式把代码传给ECS，但是不适用于ISharedComponentData,也不适合之前的内存模型
- 表面上跟值类型的IComponentData一样使用，但是ECS在内部处理上使用不用的方式，效率更低

相比值类型的IComponentData的缺陷：
- 不能使用Burst Compiler
- 不能在Job 结构中使用
- 不能使用ChunkMemory
- 需要GC

### SharedComponentData
除了用原型，还可以根据SharedComponentData指定值给具有SharedComponentData的Entity分类。

EntityManager会把具有同一SharedComponentData的entitis划分到一个Chunk。

SharedComponentData改变之后，EntityManager会把Entit移动到一个不同的Chunk，或者直接新建一个Chunk。

#### 注意
- ECS存储SharedComponentData是以Chunk为单位，而不是以实体为单位
- 因为这种内存布局，SharedComponentData对EntityQuery的效率十分友好
- EntityManager.GetAllUniqueSharedComponents
- ECS对SharedComponentData使用引用计数
- 不要轻易改变SharedComponentData，因为会导致memcpy的调用。

### SystemStateComponents
SystemStateSharedComponents与其类似，只不过内存结构不一样，shared的表现跟上文的SharedComponentData类似。

可以用来追踪资源的生命周期，而不用其他的回调。

一般的Entity Destroy流程：
- 找到Entity引用的所有Components
- 销毁找到的这些Components
- 把Entity ID 回收进行利用

但是在有SystemStateComponets的情况下，ECS不会立即回收Entity ID，这让System在Entity Destroy之后有机会进行其他操作。当SystemStateComponents销毁之后，Entity ID才会被回收。

#### 使用时机
- 当添加Component时：有其他的Components，但是没有SystemStateComponent，说明Component刚添加。
- 当删除Component时：没有对应的Components，但是有SystemStateComponents，说明Components刚刚被删除。

#### 其他
- 可见性：最好是系统外只读

### DynamicBufferComponents
DynamicBufferComponent可以把数组形式的数据跟实体联系起来。能处理大量的元素，并自动改变大小。

类似于一个SharedComponent在一个Chunk绑一个，DynamicBufferComponents也是一个Chunk绑一个。

#### 使用方式
##### 声明ElementTypes
从IBufferElementData派生

##### 绑定到Entity
- EntityMangerAddBuffer();
- 使用原型，EntityManager.CreateEntity()；
- 使用[GenerateAuthoringComponent]属性修饰IBufferElementData的派生类，然后在Editor模式直接给GameObject添加。在后台会生成从MonoBehavior派生的IntBufferElementAuthoring 。这种方式存在限制：单个C#文件中只能有一个带有这种属性的Component,且不能有另外一个MonoBehaviour；IBufferElementData只能有一个属性；不能有显示的内存分配。
- 使用EntityCommandBuffer.AddBuffer。

##### 访问Buffers
- EntityManger.GetBuffer();
- GetBufferFromEntity(): 跨Job调用；
- Entities.ForEach: 配合lambda；
- IJobChunk：BufferAccessor
- ReinterpretingBuffers: 能保证原始buffer的安全性。是对原始data的引用。


#### 其他
- Buffer reference invalidation: [StructuralChanges](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/sync_points.html#structural-changes)会导致Entities从原Chunk移动到另一个Chunk，这样会导致原来的Buffer绑定失效导致错误。

### ChunkComponentData
把存储于一个特殊Chunk里的数据应用到所有entities。

直接跟Archetype绑定。

#### 使用方式
##### 声明
从IComponentData派生。

##### 创建
可用的接口：
- AddComponent: 不能在Job里创建，也不能用EntityCommandBuffer创建
- CreateEntities:  ComponentType.ChunkComponent<T>或者ComponentType.ChunkComponentReadOnly<T> 

方式：
- Chunk中的一个entity: EntityManager.AddChunkComponentData<T>()
- EntityQuery: EntityManager.AddChunkComponentData<T>()
- EntityArchetype: EntityManager.CreateArchetype() 和 EntityManager.CreateEntity()

##### 读取
- ArchetypeChunk实例: 配合 EntityManager.GetChunkComponentData<T>()
- Chunk中的一个实例：EntityManager.GetChunkComponentData<T>()

##### 更新
接口：
- ArchetypeChunk.SetChunkComponentData： 在IJobChunk中；
- EntityManager.SetChunkComponentData：在主线程中。

方式：
- ArchetypeChunk实例：
- 通过Entity实例：

##### 删除
接口：
EntityManager.RemoveChunkComponent

### 区别
- ChunkComponentData：绑定原型，同一原型ChunkComponent的value是一样的
- SharedComponentData: 绑定的类型一样，但是同一原型下不同Chunk中SharedComponent的值是不一样的，同一Chunk中SharedComponent的值是一样的。同类型的SharedComponentData不会划分原型，但是会划分Chunk。
- DynamicComponentData: 绑定的类型一样，同一Chunk下实际上是一个List
- IComponent: 同一Chunk下的不同实体，数据都有差异。

## System
改变ComponentData的状态。

### Instantiating Systems
- Unity ECS 会在Runtime时自动找到System并初始化。可以通过添加SystemAttributes来修改系统和安排系统殊勋等；
- 一般这些系统都具有一个共同的父节点——CommponentSystemGroup：用来更新子系统；
- 没有放到ComponentSystemGroup的system会被放到一个SimulationSystemGroup中。

### System Types
- SystemBase
- EntityCommonandBufferSystem: 为其他系统提供一个EntityCommandBuffer。在一个SystemGroup的头尾各一个，用来应对StructuralChanges，就能够在一帧中使用更少的同步点。
- ComponentSystemGroup：使用嵌套的结构来为其他的系统指定更新顺序。
- GameObjectConversionSystem：编辑器模式下使用。把编辑器下的表现改成Runtime下的高效表现。

### System的生命周期
<div align="center">

![System生命周期][ECSSystemLoop]

</div>

一般情况下，在OnUpdate()中组织作业来完成大部分工作，可通过以下方式：
- Entities.ForEach
- Job.WithCode
- IJobChunk
- C# Job System

#### Entities.ForEach
##### 选择Entities
- EntityQuery:
    - OptinalComponents: WithAll/WithAny/WithNone
    - ChangeFiltering：只跟是否进行了Write Access有关系，跟访问之后有Data有没有变化没有关系。
    - SharedComponentFiltering: WithSharedComponentFilter()

##### 定义ForEach函数
基本原则
- 参数最高8个，高于8个需要自定义委托
- 对于内置的委托，参数传递需要遵循一定顺序：值类型；ref 类型；in 只读类型

###### 自定义委托
entity, entityInQueryIndex,nativeThreadIndex这几个参数一定要在自定义委托的参数列表中定义，顺序不限，但是不要加ref或者in。
    - Entity entity : 参数名可变
    - int entityInQueryIndex: 
    - int nativeThreadIndex: 

#### Job.WithCode
- Job.WithCode 传入的lambda function 无法传参，只能用CaptureLocalVar的方式获取参数。
- Schedule的使用存在限制：
    - CapturedVar必须声明为NativeArray/NativeContainer/[BittableType](https://docs.microsoft.com/en-us/dotnet/framework/interop/blittable-and-non-blittable-types)(类似值类型)
    - 即使只有一个值，也要用NativeArray返回

##### 执行方式
- Schedule：background
- Run：main thread

#### IJobChunk
通过Chunk来迭代数据。在每个Chunk内，一个一个的修改entity的相关数据。

优势：
- 更加灵活跟清楚
- 单个Chunk可以通过Archetype.Has<T>()的方式对Chunk进行手动过滤

##### 步骤
- 创建EntityQuery
- 定义Job结构，必须包含ArchetypeChunkComponentType objects的成员，让job能直接访问，并设置读写权限
- 初始化Job，在OnUpdate()中 Schedule job
- 在Execute()中对job想要操作的NativeArray进行处理： 这个时候，chunkIndex == jobIndex

##### 其他
- change Filter：
    - 默认的change filter只支持最多2个
    - 自定义支持无上限： ArchetyepeChunk.Dichange() 和LastSystemVersion。LastSystemVersion需要在Onupdate中赋值。
- Job的初始化生成要在每帧都进行，不能存成成员变量。

#### Manual Iteration
- IJobPareallelFor
- EntityManager.GetAllEntities()/EntityManager.GetAllChunks()

### System Update顺序
#### SystemOrderingAttributes
- UpdateInGroup：如果没加，就会被添加到World's SimulationSystemGroup
- UpdateBefore 和 UpdateAfter: 针对同一组内
- DisableAutoCreation : 在默认world初始化的时候不创建，必现显式创建和更新。

#### DefaultSystemGroups
主要分3类：

<div align="center">

![DefaultSystem][ECSDefaultSystem]

</div>

#### Mult Worlds
##### ICustomBootstrap
work flow：
- 创建一个World和对应的最高层的几个组
- 对于List中的system types：
    - 找到对应的组别
    - 如果找到了使用group.AddSystemToUpdateList()
    - 如果没有找到，就添加这个system type到一个list中
- 对最高层的groups调用group.SortSystemUpdatedList()
    - 可以选择将他们添加到默认world groups
- 将没有找到组别的 system list传给DefaultWorldInitialization进行初始化

> ECS 通过反射查找ICustomBootstrap

### Job Dependencies
简单来说，就是job的数据依赖。

需要注意的是：
- 如果使用Entities.ForEach 或者Job.WithCode 来做job，则需要手动处理依赖。
- 如果数据通过NativeArray进行传递，那么也需要手动处理依赖。
- Structural Changes 会导致对ComponentData的直接饮用失效，要谨慎处理。

### 查找data
#### 在系统中entity data
- 一般情况： GetComponent
- dynamic buffers: 需要先获取BufferFromEntity

#### 在IJobChunk中查找entity data
接口：
- ComponentDataFromEntity: 尽量保持Readonly
- BufferFromEntity：

### EntityCommandBuffers
主要解决的问题：
- 在job中无法访问EntityManager
- 进行Structural Change，这导致创建了一个新的同步点，必须等到所有的job都完成。

EntityCommandBuffer就是将在不同环境下的EntityManager相关命令给收集起来，在主线程中的合适时机进行处理。

## Sync Points 同步点
指在程序执行过程中，需要等其他锁有scheduled的job完成的一个时间点。

会限制在某一时间调用所有worker thread的能力。

所以要尽量避免产生同步点。

产生原因：当有其他的job在操作component数据的时候，你不能对当前的一些数据进行安全操作。

### Structural Changes
是Sync Points产生的主要原因。

产生的方式：
- 创建/销毁entities
- 添加/移除components到entity上
- 改变Shared Components的值（有可能导致chunk的重新划分）

大致上说，一切会改变实体原型或者改变chunk内实体顺序的操作都是一个structural change。

> structural change 只能在主线程上产生。

产生的影响：
- 产生Sync Points
- 导致对Component Data的直接引用失效，比如： DynamicBuffer、ComponentSystemBase.GetComponentDataFromEntity等

### 避免SyncPoints的产生
- 用EntityCommandBuffer来延迟执行一些操作
- 把会产生Structural changes的system放到一起进行Structural Changes操作

## Write Group
提供一种机制，能让一个系统能去影响另一个，即便没有修改其他系统的权限。

Write Group实际上是确定系统中一个Component作为另一个Component过滤的子集，可以使用WriteGroupFilter进行过滤，实现一种对内可见，对外不可见的状态（对Creator可见，对User不可见的状态）。

## Versions and Generations
在ECS中用到了多种Version Num，他们通常是32位signed int。只要在C#中定了signed int的溢出，那么Version num就默认是一直增加的。

- EntityId.version : 随着entity的销毁而增加。如果跟EntityManager的Version不一致，则代表entity已经被销毁
- World.version : 随着EntityManager的销毁而增加
- EntityDataManager.GlobalVerison : 在每个job component system更新之前。
- System.LastSystemVersion : 在每个job component system 更新之后。
- Chunk.ChangeVersion : 每次Chunk被写访问的时候
- EntityManager.m_ComponetTypeOrderVersion[] : 针对non-shared component type, 随着每次对应type component的iterator失效的时候增长，也就是每次改变了这个实体类型的存储数组/顺序的时候。
- SharedComponentDataManager.m_sharedComponentVersion[] : 如果带有SharedComponentData的chunk出现了structural change，就会更新这个version num

# 实践
## 架构

<div align="center">

![架构][GamePlayStructure]

</div>

## GameObject转换
条件：
- 有ConvertToEntity MonoBerhaviour Component
- SubScene的一部分
- 能转换的GameObject的子GameObject

> 前两种的区别： Subscene的数据已经序列化到了硬盘上，runtime加载的时候会很快；ConvertToEntity这种还需要ECS每次在runtime的时候将GameObjects进行转换。

可以通过IConvertGameObjectToEntity进行自定义的转换。

## 生成ComponentData
### IComponentData
添加[GenerateAuthoringComponent]属性，背地做了如下工作：自动生成了一个包含component 公共域的Monobehaviour类，提供了一个Conversion方法来把这些域转换成Runtime的ComponentData。

但是有以下限制：
- 单个C#文件里面只能有一个AuthoringComponent，并且这个C#不能有其他的Monobehaviour。
- ECS只反射公共域，并且跟component里面有相同的name
- ECS把IComponentData中entity type的域跟Monobehaviour中的域对应起来，并把赋值的GameObjects或者Prefab转换成域中引用的Prefab

### IBufferElementData
[GenerateAuthoringComponent]属性也可以添加给IBufferElementData

但是有以下限制：
- 单个C#文件里面只能有一个AuthoringComponent，并且这个C#不能有其他的Monobehaviour。
- IBufferElementData authoring component只能包含1个域
- IBufferElementData 不能自动生成有隐式内存布局的类型（不受Chunk管制的类型？）

[ECSMemoryManagement]: ./ECSMemoryManagement.jpg
[ECSSystemLoop]: ./SystemLoop.jpg
[ECSDefaultSystem]: ./ECSDefaultSystem.jpg
[GamePlayStructure]: ./GamePlayStructure.jpg

