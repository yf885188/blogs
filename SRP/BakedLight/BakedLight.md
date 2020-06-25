<!-- TOC -->

- [1. 烘焙静态光](#1-烘焙静态光)
    - [1.1. 烘焙的相关设置](#11-烘焙的相关设置)
    - [1.2. LightMap](#12-lightmap)
    - [1.3. Light Probes](#13-light-probes)
    - [1.4. LPPV (Light Probe Proxy Volumes)](#14-lppv-light-probe-proxy-volumes)
        - [1.4.1. 跟Light Probes的差异](#141-跟light-probes的差异)
        - [1.4.2. 主要流程](#142-主要流程)
    - [1.5. Meta Pass](#15-meta-pass)
    - [1.6. Baked Emission](#16-baked-emission)
    - [1.7. Baked Transparence/Clip](#17-baked-transparenceclip)
    - [1.8. GPU Instancing用于Light Probe和LPPV](#18-gpu-instancing用于light-probe和lppv)
        - [1.8.1. 用于Light Probe](#181-用于light-probe)
        - [1.8.2. 用于LPPV](#182-用于lppv)

<!-- /TOC -->

# 1. 烘焙静态光
## 1.1. 烘焙的相关设置
- 设置灯光渲染类型
- 设置Object的是否收到GI影响：Static;ContributeGI;ReceiveGI;
- 设置Lighting Setting：LightMap 相关;

## 1.2. LightMap
主要流程：
>CPU 设置相关参数->GPU管线相关设置->GPU shader转换LightMap UV->GPU shader 采样LightMap并进行混合

## 1.3. Light Probes
主要流程：
>设置light probes -> GPU 管线设置->GPU shader 转换球谐函数相关参数 ->GPU shader 采样light probes

## 1.4. LPPV (Light Probe Proxy Volumes)
### 1.4.1. 跟Light Probes的差异
> Light Probes 主要用于小型动态物体——指Light Probes构建的3D区域能很容易包裹小型动态物体的活动范围。
### 1.4.2. 主要流程
> 给相关Object添加LPPV component -> GPU 管线设置->GPU shader 转换相关参数 ->GPU shader 采样light probes

## 1.5. Meta Pass
> 没有Meta Pass材质的Object，在做GI的时候不会把它们表面反射等因素加入到考虑范围内，因此需要加入Meta Pass.

需要注意：
- Meta Pass的处理实际上是在Screen Space下的处理，所以在顶点着色器中使用LightMap的UV进行变换
- Meta Pass中对间接光的BRDF混合跟平时的处理不太一样

## 1.6. Baked Emission
需要注意：
- 设置Material.globalIlluminationFlags不为0

## 1.7. Baked Transparence/Clip
需要注意：
- bake这些表现，需要默认有_MainTex、_Color和_CutOff三种属性

## 1.8. GPU Instancing用于Light Probe和LPPV
### 1.8.1. 用于Light Probe
- Graphics.DrawMeshInstanced相关属性设置
- 通过LightProbes.CalculateInterpolatedLightAndOcclusionProbes()设置探针位置
- 通过MaterialPropertyBlock.CopySHCoefficientArraysFrom()设置球谐函数相关data

### 1.8.2. 用于LPPV
- Graphics.DrawMeshInstanced相关属性设置
- 设置LPPV Component相关参数