<!-- TOC -->

- [1. Batching](#1-batching)
    - [1.1. Pass](#11-pass)
    - [1.2. Material](#12-material)
    - [1.3. Static Batching](#13-static-batching)
    - [1.4. Dynamic Batching](#14-dynamic-batching)
    - [1.5. SRP Batching](#15-srp-batching)
        - [1.5.1. 具体原理](#151-具体原理)
        - [1.5.2. 设置](#152-设置)
        - [注意事项](#注意事项)
    - [1.6. GPU Instancing](#16-gpu-instancing)
    - [1.7. Batcher 限制](#17-batcher-限制)

<!-- /TOC -->

# 1. Batching
## 1.1. Pass
一个Shader里面通常包含多个Pass，每个Pass表示为了实现一个效果需要进行的操作，包含一个完整的vs和fs，最后的目标也是屏幕坐标系中的fragment。
## 1.2. Material
渲染的物体至少绑定一个材质。

一个材质可以看做是mesh信息、texture信息和多个渲染操作（也即render pass）的集合。
## 1.3. Static Batching
如果场景中手动设置了物体的Static属性，那么在正式渲染之前就会对场景中具有相同材质的Static Gameobject进行Static Bathcing，内部实现其实就是mesh等信息进行合并成一个巨大的mesh进行绘制。
## 1.4. Dynamic Batching
原理其实跟[Static Batching](##13-static-batching)一样，只不过是在u3d启用Dynamic Batching的情况下，在运行场景的时候，会动态的将场景中材质相同的Gameobject进行合批。
## 1.5. SRP Batching
### 1.5.1. 具体原理

参考[官网SRP Batcher](https://docs.unity3d.com/Manual/SRPBatcher.html)
<div align=center>

![SRP Batcher vs Standerd Batcher][SRPBatcherProcess]

***SRP Batcher vs Standerd Batcher***
</div>

>通过图示的比较，可以看出，Shanderd Batcher更新的基本单位是Material，一旦Material发生改变，就需要重新**更新和bind*所有的***Material信息和Object信息。SRP Batcher的的更新的基本单位是Shader，只有在Shader Variant改变的情况下才重新绑定Material信息。所以SRP Batcher要求在场景中尽可能使用同一Shader的不同Material，因此也采用了一种集群的方式为Material设置了persistent mem。
<div align=center>

![SRP Batcher Data Update Process][SRPBatcherDataUpateProcess]
***SRP Batcher Data Update Process***
</div>

上图是SRP Batcher的工作流，有几点需要注意：
- 材质数据和Object属性信息在GPU内存中是连续的；
- 材质数据不发生改变的时候，SRP Batcher 不会主动设置和更新材质数据到GPU mem。而是使用一种特定的方式快速更新一些Unity Engine属性；
- 相比传统的方式，材质更新和Oject属性信息的更新不在同步。

为了正确的使用这种工作流，有些额外的要求：
- object必须使用mesh，不包含particle 和 skinned mesh
- Shader要兼容SRP Batcher： 内置的Unity Engine Props 必须在*UnityPerDrawCBUFFER*中；所有的Material Props必须在*UnityPerMaterial CBUFFER*中。

### 1.5.2. 设置
> GraphicsSettings.useScriptableRenderPipelineBatching = true;

### 注意事项
> 在时机操作过程中发现，每次合批存在数量上限，超过数量上限的情况，单次合批上限为256，超过之后会进行平均分配

## 1.6. GPU Instancing
GPU Instancing 的使用条件：
- 必须使用mesh，但不包括skinned mesh
- 只有有相同mesh和相同material的Gameobject才能被合批
- 使用per-instance data 来设置不同的Material属性值。
>此类属性值必须在Shader中必须以下列形式定义

```
UNITY_INSTANCING_BUFFER_START(Props)
    UNITY_DEFINE_INSTANCED_PROP(fixed4, _Color)
UNITY_INSTANCING_BUFFER_END(Props)
```
>在传值的时候必须以MaterialPropertyBlock的方式设置

```
props.SetColor("_Color", new Color(r, g, b));  
renderer = obj.GetComponent<MeshRenderer>();
renderer.SetPropertyBlock(props);
```

## 1.7. Batcher 限制
u3d的合批有很多限制：
1. 非统一Scale(*各个维度的scale变化不是一致的*)的Gameobject不行；
2. Static Batching 和 Dynamic Batching 的优先级都要高于GPU Instancing,前两者激活的情况下，GPU Instancing不会启用。但是SRP Batching和GPU Instancing可以一起启用。

[SRPBatcherProcess]: ./SRPBatcherProcess.png
[SRPBatcherDataUpateProcess]: ./SRP_Batcher_Data_Update_Process.png