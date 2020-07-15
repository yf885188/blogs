# 目录结构
<div align="center">

![ShaderTOC][ShaderTOC]

</div>

Shader中引用：
- Packages/com.unity.render-pipelines.universal/Shaders/
- Packages/com.unity.render-pipelines.core/ShaderLibrary/

# 标准材质基本属性
- Surface 选项：
  - Workflow Mode : 工作流，Metallic 和 Specular。跟Fresnel基础表面反射系数有关。
  - Surface Type ：Opaque和Transparent。跟RenderQueue、Blend、Clip等有关。
  - RenderFace ：Front、Back和both。Cull设置。
  - AplhaClipping
  - ReceiveShadows
- Surface输入：
  - BaseMap：颜色和Map二选一
  - MetallicMap：数值和Map二选一
  - Smoothness：数值或者Source的来源中二选一。表面光泽度，跟微表面理论有关。
  - Source:MetallicMap的Alpha通道或者BaseMap的Alpha通道
  - NormalMap:
  - OcclusionMap:环境遮蔽视图
  - Emission:颜色和Map二选一。可以选择HDR颜色跟Map进行混合
  - Tilling：
  - Offset：
- 高级选项：
  - SpecularHighlights
  - EnvironmentReflections
  - EnableGPUInstancing
  - Priority

跟Built-in材质的区别：
- 去掉了HeightMap
- 去掉了DetailMask
- 增加了Emission贴图
- 没有第二纹理的相关设置
- 没有双面GI

# 五个基本Pass
- UniversalForward：正常渲染pass
- ShadowCaster： 阴影图渲染pass
- DepthOnly： 深度图渲染pass
- Meta：跟GI有关
- Universal2D

## UniversalForward Pass 工作流

<div align="center" >

```flow
st=>start: 入口
op_v_instance_init=>operation: [顶点着色器]初始化GPU_Instancing_ID相关
op_v_stereo_init=>operation: [顶点着色器]初始化Stero相关
op_v_ws_info_init=>operation: [顶点着色器]初始化Position、Normal、View_Dir、Fog_Factor等
op_v_lm_Pre_GI_init=>operation: [顶点着色器]计算LightMap和SH相关
op_v_shadow_init=>operation: [顶点着色器]计算Shadow UV
op_f_instance_setup=>operation: [片元着色器]设置GPU_Instancing_ID相关
op_f_stereo_setup=>operation: [片元着色器]设置Stero相关
op_f_surface_init=>operation: [片元着色器]初始化Surface结构
op_f_inputdata_init=>operation: [片元着色器]初始化InputData
                            这其中比较重要的是对一些GI预处理的采样
op_f_brdf_init=>operation: [片元着色器::PBR]初始化BRDF数据
op_f_mix_realtime_baked_gi=>operation: [片元着色器::PBR]混合GI和Realtime的阴影/灯光效果
                                      跟LightMap、ShadowMask有关。
op_f_final_GI=>operation: [片元着色器::PBR]结合BRDF得到GI在表面的最后结果
op_f_realtime_main_col=>operation: [片元着色器::PBR]根据BRDF和主光源信息计算PBS的效果
                                正常包括GGX的法线分布函数、GGX的几何衰减和Fresnel表现。
                                但是这里用的是Minimalist CookTorrance BRDF
op_f_realtime_add_col=>operation: [片元着色器::PBR]计算Additive_lights的PBS光照效果
op_f_vertex_light_col=>operation: [片元着色器::PBR]顶点光颜色
op_f_emission_light_col=>operation: [片元着色器::PBR]自发光颜色
op_f_fog_col=>operation: [片元着色器]混合fog

e=>end: 输出像素值
st->op_v_instance_init->op_v_stereo_init->op_v_ws_info_init->op_v_lm_Pre_GI_init->op_v_shadow_init
op_v_shadow_init->op_f_instance_setup->op_f_stereo_setup->op_f_surface_init->op_f_inputdata_init
op_f_inputdata_init->op_f_brdf_init->op_f_mix_realtime_baked_gi->op_f_final_GI->op_f_realtime_main_col->op_f_realtime_add_col->op_f_vertex_light_col->op_f_emission_light_col->op_f_fog_col->e

```

</div>


# 相关文件分类
| 文件名 | 作用|
| :---: | :--- |
| LitInput | 记录一些材质相关属性，并把属性转换为SurfaceData |
|Core | Unity内置宏的转换；相关结构、宏的定义；<br>位置、法线相关结构的定义和转换 |
|CommonMaterial|Smoothness、各向异性等参数相关操作 |
|SurfaceInput| SurfaceData结构体的定义及其相关参数的转换|
|Packing|对各种数据进行进行解码或者其他处理，比如：UnpackNormal等|
|Lighting|根据是否有LightMap定义不同的宏，及LightMap相关操作接口;<br>定义Light结构及相关操作接口；<br>BRDF结构及相关操作接口,比如：InitializeBRDFData；<br>顶点着色器和片元着色器中的相关操作函数，包括一些经典模型的计算函数，比如：UniversalFragmentPBR、UniversalFragmentBlinnPhong等。|
|Common| 常用的顶点着色器和片元着色器的输入输出结构，比如：Attributes、Varyings等；<br>一些辅助函数：TransformFullscreenMesh、VertFullscreenMesh、GetLuminance、ApplyVignette、ApplyTonemap、ApplyColorGrading、ApplyGrain、ApplyDithering|
|Color|不同颜色空间的转换接口；<br>相关颜色参数的计算，比如：ScotopicLuminance；<br>ToneMap相关操作；<br>LutMap相关操作|
|ACES|ACES相关|
|EntityLighting|GI相关的操作，包括：采样LightMap、SH采样计算等|
|ImageBasedLighting| IBL，[参考](https://zhuanlan.zhihu.com/p/26449508)|
|Shadows|阴影的处理，包括：采样ShadowMap、处理级联阴影混合、阴影渐变等|
|ShadowSamplingTent| 级联阴影采样的一些接口|



[ShaderTOC]: ./ShaderTOC.jpg