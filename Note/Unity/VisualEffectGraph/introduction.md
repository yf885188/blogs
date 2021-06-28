[参考](https://docs.unity.cn/Packages/com.unity.visualeffectgraph@12.0/manual/index.html)

# node类型
- operator nodes
- block : 针对effect的操作
- context ： 几个block的联合

# Visual Effect Graph Assets
包含的内容：
- Graph element
- Exposed properties
- compiled shaders
- operator bytecode 

# Graphic logic
- vertical logic : processing. 流程
- horizontal logic : property. 属性

logic >> system >> context >> blocks == ops

## system
vfx的一个独立功能

## context
blocks的集合。联合之后成为system

- context的连接有限制
- 主要类型：
  - event
  - spawn


## blocks
一个block是一个独立操作，node

## operators
链接属性workflow的低等级操作。跟block说的独立操作不是一个东西。一个是针对粒子，一个是针对node，主要针对property workflow。

## properties
- 基本类型
- 基本类型转换
- 结构
- 空间相关的结构：local/world
- 性质节点：在Blackboard里定义的，graph范围内有效的性质，可以复用。

## events
- 没有输入只有输出
- 只能连到Spawn或者Initialize context上

缺省events
OnPlay/OnStop:Spawn context的默认start/stop输入

### GPU Events
根据其他粒子来spawn粒子。

### Attributes
system中元素附带的一块数据。

与property的区别：
- property用于property workflow，用于system外部
- attributes用于内部

### Subgraph
在不同Graph中可复用的资产。

分类：
- System Subgrah
- Block Subgrah
- Operator Subgrah

### BlackBoard
BlackBoard中定义的变量通常都是全局变量。

### sticky node
graph中可以写入的辅助结构，比如note。

### visual effect bounds
- 在initialize context中定义bounds
- bounds设定模式：
  - manual
  - recorded
  - automatic

#### Bounds Padding
如果使用Recorded或者automatic模式，则在Update Context中计算system bound。

不过这也导致了Bound的更新会落后于实体数据的更新。

# Visual Effect Component 
## C# Component API
### Events
- Event Attributes:
  - 复用的event attributes最好store
  - 每次send event都是发送的的event的copy
  - 使用同一Visual Effect component的visual effect graph asset绑定

## timeline
### Visual Effect Activation
- Visual Effect activation track
- Visual Effect Activation Clip
### Animating Properties
- 对暴露的Properies进行处理

## Property Binders
把场景中实体的相关信息设置到暴露的property上。

## Event Binders
场景中某些事件发生的时候用来触发visual effect的脚本。

## Output Event Handler
out event的绑定


## 在visual effect中表示复杂形状
两种方式的比较：
- SDF更吃资源，但更实用
- point cache 

### SDF
使用
- 根据SDF设置位置
- 根据SDF吸引粒子
- 模拟碰撞
- 采样SDF

生成
- SDF Bake Tool
- Houdini Volume Exporter

### Point Cache
记录particle attribute data的一种资产。

> Point relaxtation 是一个更均匀地分离点和减少重叠的处理过程。

生成
- Point Cache Bake Tool
- pCache Exporter

#### Point Cache Bake Tool
两种模式：
- bake from mesh
- bake from texture