# 植被种类
- FoliageType_Actor
- FoliageType_InstancedStaticMesh

每个类型都可以有对应的类型实体（同一张贴图、mesh等），然后包含一套transform信息，用来instance时摆放instance。

# FoliageType_InstancedStaticMesh
FoliageType_InstancedStaticMesh 使用UHierarchicalInstancedStaticMeshComponent也即HISM来管理。

UHierarchicalInstancedStaticMeshComponent::SetPerInstanceLightMapAndEditorData 对每个instance的lightmap相关参数（LightmapUVBias和ShadowmapUVBias）进行了赋值，可以推测同一type实体的instances共用lightmap。

