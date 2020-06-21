# 烘焙静态光
## 烘焙的相关设置
- 设置灯光渲染类型
- 设置Object的是否收到GI影响：Static;ContributeGI;ReceiveGI;
- 设置Lighting Setting：LightMap 相关;

## 代码
主要流程：
>CPU 设置相关参数->GPU管线相关设置->GPU shader转换LightMap UV->GPU shader 采样LightMap并进行混合
