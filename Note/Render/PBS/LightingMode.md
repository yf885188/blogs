<!-- TOC -->

- [1. 经验模型](#1-经验模型)
  - [1.1. Lambert](#11-lambert)
  - [1.2. Phong](#12-phong)
  - [1.3. Blinn-Phong](#13-blinn-phong)
- [2. PBS渲染](#2-pbs渲染)

<!-- /TOC -->

# 1. 经验模型
## 1.1. Lambert
漫反射：
$$I_{do} = I_{i} * K_d * max(cos\theta, 0)$$
- $I_{i}$ 入射光强
- $I_{do}$ 漫反射光强
- $K_{d}$ 漫反射系数
- $\theta$ 反射光线跟法线的夹角

环境光：
$$ I_{ao} = I_{ai} * K_a $$
- $I_{ai}$ 入射环境光强
- $I_{ao}$ 反射环境光强
- $K_a$ 环境光反射系数

Larmbert光照模型
$$I_{Lambertian} = I_{ao} + I_{eo} + I_{do}$$
- $I_{eo}$ 自发光

## 1.2. Phong
$$ I_{so} = K_s * I_{si} *[max(\vec(R)\cdot \vec(V), 0)]^{sh} $$

- $K_s$ 高光反射系数
- $I_{si}$ 高光强度
- $\vec(R)$ 反射光方向
- $\vec(V)$ 视线方向
- $sh$ 表面光滑度
  
$$ I_{Phong} = I_{Lambertian} + I_{so}$$

## 1.3. Blinn-Phong
$$ I_{so} = K_s * I_{si} *[\vec(H)\cdot \vec(N)]^{sh} $$
- $\vec(H)$ 为入射光线反向向量和观察向量的半角向量
- $\vec(N)$ 法线向量

$$ I_{Blinn-Phong} = I_{Lambertian} + I_{so}$$
> 无用计算反射向量，速度更快

# 2. PBS渲染
详见[PBS](./PBS.md)