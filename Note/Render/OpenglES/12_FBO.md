# 渲染到纹理
- glCopyTexImage2D/glCopyTextureSubImage2D来实现
- 使用连接到纹理的pbuffer来实现渲染到纹理

# 结构
- FBO
  - Color Attachment:包括渲染缓冲区对象
  - Depth Attachment
  - Stencil Attachment

![][FBOStructure]

[FBOStructure]: ./FBOStructure.jpg


> FBO、渲染缓冲区和纹理是不同的概念

# 渲染缓冲区与纹理作为帧缓冲区的对比
使用渲染缓冲区的原因：
- 可以使用多重采样
- 可以使用非纹理的格式存储，在某些情况下比纹理更高效

# FBO与窗口系统EGL表面的对比
- 像素归属测试
- EGL可能只支持双缓冲表面，FBO只支持单缓冲
- FBO可以实现模板和涉毒缓冲区的共享

# 渲染缓冲区的使用

# FBO的使用
FBO的相关状态
- 颜色附着点
- 深度附着点
- 模板附着点
- 完整性状态

> 每个附着点可以指定：对象类型，对象名称，纹理级别，纹理立方图面，纹理层次。

## 使用流程
- 创建FBO
- 创建渲染缓冲区
- 绑定：glFramebufferRenderbuffer(渲染缓冲区)/glFramebufferTexture2D(纹理)/glFramebufferTextureLayer(3D纹理)
- 删除

## 完整检查
- 三个附着有效
- 有效附着的宽高必须相同
- 深度和模板附着必须一致（符合模板测试通过了之后才只能深度测试的规则？）
- 渲染缓冲区附着的GL_RenderBuffer_Samples的值必须都相同

# FBO区块位传送
- glBlitFramebuffer

# FBO失效
- glInvalidateFramebuffer
- glInvalidateSubFramebuffer

# 读取Pixel和FBO
glReadPixels:
- 先调用glBindBuffer绑定到GL_Pixel_PACK_BUFFER（数据结构转换？）

# 性能提示
- 不要频繁切换
- 不要逐帧创建/删除FBO/渲染缓冲区
- 颜色附着点的纹理避免修改
- 整图渲染的时候，glTexImage2D和glTexImage3D的pixel参数设置为null，表示不使用原始数据。同样，使用原纹理的时候，使用glInavlidateFrameBuffer来使FBO失效
- 深度/模板缓冲区优先使用共享的
