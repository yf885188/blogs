<!-- TOC -->

- [1. 数据格式](#1-数据格式)
- [2. 模板测试](#2-模板测试)
- [3. 深度/模板状态](#3-深度模板状态)
- [4. 使用](#4-使用)
- [5. 通用阴影矩阵](#5-通用阴影矩阵)
- [6. 细节问题记录](#6-细节问题记录)

<!-- /TOC -->

# 1. 数据格式
格式：
- DXGI_FORMAT_D32_FLOAT_S8X24_UINT
- DXGI_FORMAT_D21_UNORM_S8_UINT

重置局部数据：
ID3D12GraphicsCommandList::ClearDepthStencilView()

# 2. 模板测试
- 模板参考值 & 掩码值
- 特定像素值 & 掩码值
以上两个通过制定的模板比较函数来确定。

# 3. 深度/模板状态
D3D12_DEPTH_STENCIL_DESC
- 深度信息相关设置
- 模板信息相关设置
  - D3D12_DEPTH_STENCILOP_DESC

# 4. 使用
- 填写D3D12_DEPTH_STENCIL_DESC，赋值给D3D12_GRAPHICS_STATE_DESC::DepthStencilState
- 设置参考值：ID3D12GraphicsCommandList::OMSetStencilRef

# 5. 通用阴影矩阵
- $L_w$=0表示平行光，反之为点光源。

# 6. 细节问题记录
- 镜面反射完了之后，要注意对模型的绕序方向进行设置。
- 镜面的物体渲染顺序：Opaque->Mirror->Reflection->Transparent。*主要*由depth是否会影响后续的渲染来确定。
- 模板值的范围由模型在RenderTarget中覆盖的像素决定（实时上并不去改写RenderTarget），单个模板值的大小由接口指定，也就是说单次模板设置的模板值都是一样的

待解决：
- CreateSampler超过3个容易crush