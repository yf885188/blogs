# static mesh component的shadowmap来源
UStaticMeshComponent::GetMeshMapBuildData:根据LODInfo来确认

前置判断：
- 是否有静态static mesh
- LOD是否与cache的兼容

参数：
FStaticMeshComponentLODInfo.MapBuildDataId

取值：
- 如果有Override Mat，直接返回Override的MapBuildData
- 有Owner
  - 有OwnerLevel且OwningWorld
    - 有ActiveLightingScenario ：返回ActiveLightingScenario->MapBuildData
    - 有OwnerLevel:返回OwnerLevel->MapBuildData