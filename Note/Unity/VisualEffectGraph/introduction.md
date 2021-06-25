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

## blocks
一个block是一个独立操作，node

## operators
链接属性workflow的低等级操作。跟block说的独立操作不是一个东西。一个是针对粒子，一个是针对node。