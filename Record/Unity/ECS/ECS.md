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

[ECSMemoryManagement]: ./ECSMemoryManagement.jpg



