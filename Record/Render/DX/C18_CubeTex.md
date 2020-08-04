<!-- TOC -->

- [1. 立方体纹理](#1-立方体纹理)
- [2. 天空盒](#2-天空盒)
- [3. 模拟反射](#3-模拟反射)
- [4. 绘制动态立方体图work flow](#4-绘制动态立方体图work-flow)
- [5. 几何着色器绘制动态立方体图](#5-几何着色器绘制动态立方体图)

<!-- /TOC -->

# 1. 立方体纹理
- 环境贴图
- 天空盒

# 2. 天空盒
- 为了减少overdraw，最后绘制天空盒
- 跟着摄像机一起移动

# 3. 模拟反射
- 镜面反射对立方体贴图的采样，都是通过反射向量来做的
  - 会导致一个问题：平滑表面同样的反射向量不同的点采样的环境贴图纹素一样。
  - 解决方案：通过代理几何体。给skybox加一个AABB。加上起始点信息的向量（原来的方式只有方向没有起点），通过与skybox的包围盒进行相交检测得到的点来确定后续取的纹素。

# 4. 绘制动态立方体图work flow
- 构建立方体图资源
  - D3D12_RESOURCE_DESC.DepthOrArraySize
  - D3D12_RESOURCE_DESC.Flags = D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET
- 分配描述符堆
- 构建描述符
  - D3D12_SHADER_RESOURCE_VIEW_DESC.ViewDimension
  - 六个面的TargetViewDesc: 
    - D3D12_RENDER_TARGET_VIEW_DESC.ViewDimension
    - D3D12_RENDER_TARGET_VIEW_DESC.Texture2DArray.FirstArraySlice
    - D3D12_RENDER_TARGET_VIEW_DESC.Texture2DArray.ArraySize
- 构建深度缓冲区：只需要构建一个，然后给6个面的render target公用
- 设置立方体图的视口与裁剪矩形
- 设置立方体师徒的摄像机：6个，fov均为90度
- 对立方体图进行绘制：分别绘制到6个Render_Target_View上

# 5. 几何着色器绘制动态立方体图
- 设置纹理数组，用于Render_Target
- 6个深度缓冲区（在几何着色器中要同时使用）
- 把设置的Render_Target和深度缓冲区到绑定到输入装配阶段
- 在几何缓冲区中通过SV_RenderTargetArrayIndex来索引多目标的渲染对象
  
优势：
- 一次性绘制了CubeMap的所有面

劣势：
- 几何着色器要处理大量的顶点数据，会降低性能
- 在几何着色器中将一个顶点处理了6次，但是在后面的光栅化阶段要剔除其中5次的处理结果，会照成大量的性能消耗。并且，在视锥体剔除阶段，我们就已经通过6个相机剔除了相当一部分多的实体。

结论：只有在绘制**动态天空盒**这种实体包围场景/摄像机必须使用动态立方体图的场合下，才有会将6次绘制降低到1次绘制带来的性能提高（因为，6个相机绘制天空盒，在光栅化阶段本来一个顶点也会绘制6次剔除5次）。