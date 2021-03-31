# 细节
- glStencilMaskSeparate:对背面和正面使用不同的模板掩码
- glScissor:剪裁测试跟光栅化时候的裁剪不一样
- glStencilFuncSeparate:也是分正反面

# MSAA
- SubPixel
- GL_SAMPLE_ALPHA_TO_COVERAGE:SubPixel的Alpha混合得到最终的alpha
- 质心采样：centroid

# MRT
- 创建和绑定FBO：glGenFrameBuffers glBindFrameBuffers
- 初始化纹理
- 纹理绑定到FBO
- 指定FBO的颜色attachment