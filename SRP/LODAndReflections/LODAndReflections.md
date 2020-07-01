# Lod Groups
## Lod Bias

## Lod Transitions 
- Cross Fade 
- Speed Tree.

## Dithering 
- CrossFade对应unity_LODFade参数；
- 两个不同level的Lod进行Fade混合时，当前层级的unity_LODFade.x大于零，下一个层级的小于零。
- transiton width 控制混合

## Animated Cross-Fading
- 跟transiton width 无关，过了阈值之后就有一个交叉变换，持续一段时间，通过 LODGroup.crossFadeAnimationDuration设置，用于全部的LOD Groups.

# Reflections
前两章的Baked Light 和 Shadow Mask 只处理了GI中的Diffuse没有处理Specular。

## 混合
注意一下当前的混合方式：

```
GI.diffuse * brdf.diffuse + GI.Specular * factor * brdf.Sepcular;
```

## Sample the Enviroment
- 采样unity_SpecCube

## Rough Reflections
BRDF.roughness 会降低Specular的Intensity，也会使Sepcular更混乱.

## Fresnel Reflection
使用 Schlick的近似处理方案
```
#define MIN_REFLECTIVITY 0.04

brdf.fresnel = saturate(surface.smoothness + 1.0 - ((1 - MIN_REFLECTIVITY) - surface.metallic*(1 - MIN_REFLECTIVITY)));
```

## 反射探针 Reflection Probes
上面的操作只采样了Skybox，是因为environment cube map中只包含Skybox。现在需要把其他的物体也加到environment cube map中。

模式：Baked; Realtime;
 
注意事项
- 根据情况防止一个/多个探针到不同的位置,并设置探针的类型和参数等
- 一般都是自动选择important probe,但是也可以用MeshRenderer.AnchorOverride进行重载，这样的话能省去探针的设置
- MeshRenderer.ReflectionProbes模式有4种。但是Blend Probes相关的两种会破坏掉SRP Batchers，所以一般只使用Skybox和Simple两种

## unity_SpecCube0_HDR
存在cube map是HDR的情况，需要注意。

