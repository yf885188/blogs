# 应用场景
- LightMap没有Shadow Max Distance的限制，但是不能渲染阴影
- 渲染的阴影不会被剔除掉，但是没有实现表现的效果
> ShadowMask使用场景:在Shadow Max Distance内用实时阴影，在Shadow Max Distance之外用ShadowMask，获得一个比较好的渲染体验。

# 烘焙阴影
## 流程
> Lighting Mode 选择ShadowMask -> ShadowMask Mode 选择 Distance ShadowMask -> 管线检测ShadowMask是否启用 -> Shadow中采样Shadow Mask 相关数据并加到GI中。

## Occlusion Probes
动态物体要享受到ShadowMask的影响，只能通过Occlusion Probes来实现。