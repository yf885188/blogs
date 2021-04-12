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

## MapBuildData
### 填充数据
- UMapBuildDataRegistry::AllocateMeshBuildData
- UMapBuildDataRegistry::InvalidateStaticLighting：看原有的资源是否保持，如果保持的话就更新到MeshMapBuildData

#### UMapBuildDataRegistry::AllocateMeshBuildData
途径：
- UInstancedStaticMeshComponent::ApplyLightMapping （InstancedStaticMesh）
- ULevel::HandleLegacyMapBuildData : GComponentsWithLegacyLightmaps上的 LegacyMeshData直接copy
- UModel::ApplyStaticLighting ：Surface Lightmap / ShadowMap 进行合图，Pack设置相关的参数，并更新受到影响的MeshMapBuildData.ShadowMap
- FStaticMeshStaticLightingTextureMapping::Apply : 不是用VT有相关数据就要更新ShadowMap, 维护一个不不相关的light列表

![][FStaticMeshStaticLightingTextureMappingMappingWorkFlow]

[FStaticMeshStaticLightingTextureMappingMappingWorkFlow]: ./images/FStaticMeshStaticLightingTextureMappingMappingWorkFlow.jpg

- FLandscapeStaticLightingTextureMapping::Apply ： 跟上面的类似，就是加了植被
