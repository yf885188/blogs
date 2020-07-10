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
- SharedComponentData: 绑定的类型一样，但是同一原型下不同Chunk中SharedComponent的值是不一样的，同一Chunk种SharedComponent的值是一样的
- DynamicComponentData: 绑定的类型一样，同一Chunk下实际上是一个List
- IComponent: 同一Chunk下的不同实体，数据都有差异

[ECSMemoryManagement]: ./ECSMemoryManagement.jpg



