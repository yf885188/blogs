# 纹理使用过程
- 创建：glGenTexture
- 绑定：glBindTexture
- 加载：glTexImage2D/glTexStorage2D/glTexSubImage2D
- 删除：glDeleteTexture

## 其他
- glPixelStorei:解包对齐
- glReadPixels
- glTexParameter

## 纹理过滤+Wrap

## 纹理调配(Swizzle)
改变某一个格式的映射方式。
> 使用glTexParameter来完成

## LOD
平滑计算mip等级的一种方式，防止间接失真。

## PCF

# 纹理格式
## 规范化纹理格式
- 规范化：采样之后的值位于[0.0, 1.0]/[-1.0, 1.0]范围内

> 不同格式对于渲染和过滤的支持不一样

## 浮点纹理格式

## 整数纹理格式

## 共享指数纹理格式
通常用于HDR。存在一个解码过程。

## sRGB纹理格式
非线性颜色空间。使用sRGB颜色空间，大概是个幂函数曲线。

正确使用方式：
纹理使用sRGB->shader中读取之后转换为线性颜色空间->渲染到一个sRGB目标(颜色自动的转换为sRGB)

> - 可以使用pow(value, 2.2)进行近似的sRGB->线性空间的转换
> - 使用pow(value, 1/2.2)进行一个近似的逆转换

## 深度纹理

## 着色器中使用纹理
- glActiveTexture
- glBindTexture
- 绑定sample

使用：
- 立方体纹理
- 3D纹理
- 2D纹理数组

# 压缩纹理
- ETC2
- EAC

API：
- glCompressTexImage2D
- glCompressTexImage3D

# 纹理子图像规范