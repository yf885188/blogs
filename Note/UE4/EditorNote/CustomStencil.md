# 简介
不同于系统使用的StencilDetphBuffer，类似于DepthOnlyPass新建了一个单独的管线和RT来做。

# UE4中的实现
资源：
- FSystemTextures.StencilDummySRV : FSystemTextures.StencilDummySRV 

资源设置接口：
- SetupSceneTextureUniformParameters 

封装的参数：
- SceneTextureParameters