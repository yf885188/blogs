调用UHT，进行文件的合并/反射等等

# 分类
- BuildMode
- RebuildMode
- CleanMode
...

## BuildMode
通过Makefile的相关信息来调用UHT，并根据相关结果，过滤需要进行的Action，并对Action进行优化。

- UnrealBuildTool.BuildMode.Build ： 主要逻辑
- ExternalExecution.ExecuteHeaderToolIfNecessary
- GetActionsForTarget ：为平台编译提取相关动作
- ActionGraph.ExecuteActions : 执行动作。并为Action选定编译的平台。 xxx::ExecuteActions
  - XGE : 最后走到UnrealBuildTool.XGE.ExecuteTaskFile
  - Distcc
  - SNDBS
  - ...

> action的生成其实都依赖UHT的结果

### UnrealBuildTool.ExternalExecution.ExecuteHeaderToolIfNecessary 
判断是否需要跑UHT：（判断有先后顺序）
- 没有UHT程序
- 强制生成BuildConfiguration.bForceHeaderGeneration
- AreGeneratedCodeFilesOutOfDate ： 判断生成的代码文件是否过期
  - 模块只读
  - GeneratedCodeFile的目录不存在
  - UHT的TimeStamp文件不存在
  - UHT时间戳比GeneratedCodeFile的TimeStamp文件时间戳要新
  - CoreUObject的TimeStamp文件的时间戳要比UHT的TimeStamp文件时间戳要新
  - 对应的ModuleRulesFile不存在
  - 对应的ModuleRulesFile的时间戳要比GeneratedCodeFile的Timestamp文件时间戳要新
  - 查看TimeStamp文件里的文件
    - 数量发生变化
    - 不在现在模块的头文件里面
  - 模块的头文件比GeneratedCodeFile的Timestamp文件时间戳要新
  - 如果!BuildConfiguration.bUseUBTMakefiles && (bIsGatheringBuild || !bIsAssemblingBuild)，判断头文件所在的文件夹是否比GeneratedCodeFile的Timestamp文件时间戳要新
- UHT版本是否正确
- AreExternalDependenciesOutOfDate：额外依赖是否过期

### GetActionsForTarget
- UnrealBuildTool.ActionGraph.GetActionsToExecute 
  - UnrealBuildTool.ActionGraph.GatherAllOutdatedActions 
    - 生成的中间文件等是否过期
    - PrerequisiteActions是否过期 ：过期之后，判断库的过期（会受配置影响）
    - PrerequisiteItems是否过期
    - DependencyListFile是否过期


# 其他
- UBT的配置：BuildConfiguration.xml有坑，每个开关上面还有一层配置节点

![][BuildConfigSample]

[BuildConfigSample]: ./images/BuildConfigSample.jpg