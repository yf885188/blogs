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