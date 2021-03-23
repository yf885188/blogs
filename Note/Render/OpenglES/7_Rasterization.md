# spirit
- spirit的坐标原点是(左，上)，但是opengl es的坐标原点是(左，下)

# 绘制图元
- 非实例化方式
  - glDrawArrays:元素索引按顺序排列
  - glDrawElements：有indices
  - glDrawRangeElements：有范围
- 实例化方式
  - glDrawArraysInstanced
  - glDrawElementsInstanced

## 图元重启
- 在索引列表中插入特殊索引来重启一个索引的绘制状态（打断绘制索引）。

## 驱动顶点
- 不插值时，确定使用哪个顶点值用于FS中

## 实例化渲染
- glDrawArraysInstanced
- glDrawElementsInstanced

实例属性的读取：
- 使用glVertexAttribDivisor来设定是按照顶点读取顶点属性，还是按照图元来读取顶点属性。
- 使用gl_InstancedID作为顶点着色器中的缓冲区索引，以访问每个实例的数据。

## 性能提示
- 通过图元重启来连接不连续的图元，在旧的API中使用三角形退化的方式进行
- 连接的时候需要注意三角形的顶点顺序（为了保证都是顺时针/逆时针）

# 图元装配

![][CoordinatorSystem]

[CoordinatorSystem]: ./images/CoordinatorSystem.jpg

- 透视分割：进行了透视除法，得到NDC坐标系坐标。
- 视口变换：得到屏幕坐标

# 光栅化
- cull face
- bias

# 遮挡查询