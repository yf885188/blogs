[链接](https://mikelis.net/garbage-collection-in-ue4-a-high-level-overview/)

# GC的条件
- object不在被使用property declaration的reflection system引用
- object的指针必须被存在container class中
- object不能有强指针类型：shared pointer，shared references,unique pointer或者strong pointers
- object创建的代码已经执行完
- object不是GC graph中 root set的一部分
- 从UObject派生

# 潜规则
- Actor 和ActorComponent 不用考虑GC
- 载入新level的时候，会把旧level的相关东西都destroy掉
- invalid object的指针不一定是nullptr，用isvalid判断
- 不是从UObject class派生的class都不会走GC，保证class都是从UE4的自带class派生的，至少从UObject

# 实现
## GC Graph

![][GCGraph]

[GCGraph]: ./images/GCGraph.jpg

- untouchable list: 要被GC的list