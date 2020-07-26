<!-- TOC -->

- [1. 数据格式](#1-数据格式)
- [2. 模板测试](#2-模板测试)
- [3. 深度/模板状态](#3-深度模板状态)
- [4. 使用](#4-使用)

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