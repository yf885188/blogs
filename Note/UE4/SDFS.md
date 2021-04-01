# 存储结构
- FTextureMappingStaticLightingData ： 所有光照相关的存储结构

# 烘焙接口
- FStaticLightingSystem::CalculateDirectSignedDistanceFieldLightingTextureMappingTextureSpace
- FLightmassSolverExporter::ExportResults : 输出最后的结果

# 实际结构
```
struct FSignedDistanceFieldShadowSampleData
{
    /** Normalized and encoded distance to the nearest shadow transition, in the range [0,1], where .5 is the transition. */
    float Distance;
    /** Normalized penumbra size, in [0,1] */
    float PenumbraSize;
    /** True if this sample maps to a valid point on a surface. */
    bool bIsMapped;
};
```

# 获取InvUniformPenumbraSizes的方式
```
const FShadowMapInteraction ShadowMapInteraction = LCI ? LCI->GetShadowMapInteraction(FeatureLevel) : FShadowMapInteraction();
if (ShadowMapInteraction.GetType() == SMIT_Texture)
{
    Parameters.ShadowMapCoordinateScaleBias = FVector4(ShadowMapInteraction.GetCoordinateScale(), ShadowMapInteraction.GetCoordinateBias());
    Parameters.StaticShadowMapMasks = FVector4(ShadowMapInteraction.GetChannelValid(0), ShadowMapInteraction.GetChannelValid(1), ShadowMapInteraction.GetChannelValid(2), ShadowMapInteraction.GetChannelValid(3));
    Parameters.InvUniformPenumbraSizes = ShadowMapInteraction.GetInvUniformPenumbraSize();
}
else
{
    Parameters.ShadowMapCoordinateScaleBias = FVector4(1, 1, 0, 0);
    Parameters.StaticShadowMapMasks = FVector4(1, 1, 1, 1);
    Parameters.InvUniformPenumbraSizes = FVector4(0, 0, 0, 0);
}
```