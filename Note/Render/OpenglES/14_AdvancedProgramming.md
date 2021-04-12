# 发现贴图
- 使用切线空间而不是世界空间

# 使用变换反馈的GPU粒子
- 两个pass
- 第一个pass生成随机粒子数组缓冲区
- 第二个pass用于渲染
- 帧间用ping pong的方式更新缓冲区

相关API：
- glTransformFeedbackVaryings