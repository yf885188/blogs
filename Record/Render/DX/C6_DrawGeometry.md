<!-- TOC -->

- [1. 顶点结构和输入布局](#1-顶点结构和输入布局)
- [2. 顶点缓冲区](#2-顶点缓冲区)
  - [2.1. 创建](#21-创建)
  - [2.2. 数据更新](#22-数据更新)
  - [2.3. 绑定](#23-绑定)
  - [2.4. 绘制](#24-绘制)
- [3. 索引缓冲区](#3-索引缓冲区)
  - [3.1. 创建](#31-创建)
  - [3.2. 绘制](#32-绘制)
  - [3.3. 合并绘制](#33-合并绘制)
- [4. 顶点着色器](#4-顶点着色器)
  - [4.1. HLSL](#41-hlsl)
  - [4.2. 输入布局描述符与输入签名](#42-输入布局描述符与输入签名)
- [5. 像素着色器](#5-像素着色器)
- [6. 常量缓冲区](#6-常量缓冲区)
  - [6.1. 创建](#61-创建)
  - [6.2. 更新数据](#62-更新数据)
  - [6.3. 常量缓冲区描述符](#63-常量缓冲区描述符)
  - [6.4. 根签名和描述符表](#64-根签名和描述符表)
- [7. 编译着色器](#7-编译着色器)
  - [7.1. 离线编译](#71-离线编译)
- [8. 光栅器状态](#8-光栅器状态)
- [9. 流水线状态对象 Pipeline State Object(PSO)](#9-流水线状态对象-pipeline-state-objectpso)
- [10. 问题记录](#10-问题记录)
  - [10.1. 内存对齐问题](#101-内存对齐问题)

<!-- /TOC -->

# 1. 顶点结构和输入布局
- 顶点结构: GPU RP中用到的数据。
- 输入布局描述: 给GPU定义如何使用顶点结构的结构。
  - D3D12_INPUT_LAYOUT_DESC
  - D3D12_INPUT_ELEMENT_DESC
    - SemanticName：语义名
    - SemanticIndex: 同一语义不同的多个索引
    - Format：顶点元素格式
    - InputSlot：输入槽,向顶点装备阶段传递顶点数据
    - AlignedByteOffset: 单个属性的偏移量
    - InputSlotClass
    - InstanceDataStepRate

# 2. 顶点缓冲区
## 2.1. 创建
- 先填写D3D12_RESOURCE_DESC结构体来描述缓冲区资源
- 调用ID3D12Device::CreateCommittedResource创建ID3D12Resource
> 提供了封装类CD3D12_RESOURCE_DESC来提供一些简化的资源创建接口

## 2.2. 数据更新
- 创建一个中介位置的上传缓冲区：资源先从内存上传到上传缓冲区，然后再复制到顶点缓冲区。
- 通过D3D12_SUBRESOURCE_DATA这个结构来确定要复制到顶点缓冲区的数据。
  
## 2.3. 绑定
- 创建一个顶点缓冲区视图来绑定，使用D3D12_VERTEX_BUFFER_VIEW结构来定义
- 顶点缓冲区视图要跟输入槽绑定，而输入槽直接为管线的起点顶点装配阶段服务

## 2.4. 绘制
- 设置图元：cmdList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_XXXX)
- 具体绘制：ID3D12GraphicsCommandList::DrawInstanced()
  - 顶点的数量
  - 实例的数量
  - 顶点缓冲区第一个被绘制顶点的索引
  - 实例开始的索引

# 3. 索引缓冲区
大体上跟顶点缓冲区一致
## 3.1. 创建
使用D3D12_INDEX_BUFFER_VIEW结构创建

## 3.2. 绘制
ID3D12GraphicsCommandList::DrawIndexInstanced()

## 3.3. 合并绘制
index的偏移

# 4. 顶点着色器
## 4.1. HLSL
> 顶点着色器中并没有进行透视除法，后期通过硬件实现

## 4.2. 输入布局描述符与输入签名
在创建ID3D12PipelienState对象时就需要指定。

> 可以不一致，但是仅限描述符比签名多、某些类型不匹配等情况。

# 5. 像素着色器
- 深度测试等对像素着色器的影响

# 6. 常量缓冲区
## 6.1. 创建
常量缓冲区随着CPU每帧更新一次，所以把常量缓冲区创建到一个上传堆而非默认堆中。一般都是硬件最小分配空间(256B)的整数倍。

GPU端的两种表现：
```
//第一种表示
cbuffer cbPerObject : register(b0)
{
    int m;
}
访问：
int n = m;

//第二种表示
stuct ObjectConstants
{
    int m;
};
ConstantBuffer<ObjectConstants> gObjConstants : regester(b0);
访问：
int n = gObjConstants.m;
```
> register(*#)中，*表示寄存器传递的资源类型：
    - t：着色器资源视图
    - s: 采样器
    - u: 无序访问视图
    - b：常量缓冲区视图

CPU端的创建：
- D3D12_CONSTANT_BUFFER_VIEW_DESC
- 调用ID3D12Device::CreateConstantBufferView

## 6.2. 更新数据
Map和UnMap。
跟把上传堆跟CPU中的一种内存进行Map，每次更新的时候跟新这块内存的数据，就能更新到上传堆的数据。

## 6.3. 常量缓冲区描述符
- D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV：描述符堆的类型

## 6.4. 根签名和描述符表
根签名：在执行绘制命令之前，定义哪些资源需要被绑定到渲染流水线上，并会被映射到着色器对应的输入寄存器中。

流程：
- 创建根签名：
  - 使用根常量、根描述符或者描述符表来生成一个根参数
  - 根据根参数创建一个CD3DX12_ROOT_SIGNATURE_DESC结构的描述布局
  - 把描述布局序列化
  - 根据序列化后的描述布局创建根签名
- 根据根签名进行资源绑定
  - 用命令列表设置根签名和CBV
  - 根据绑定好的根签名来设置需要绑定的描述符表

<div align="center">

![RootSignature][RootSignatureMem]

</div>

> 根签名与描述符堆并不一定是一一对应的关系：
> - 根签名更多的是描述一种内存布局，跟shader里面全局变量所在的寄存器更相关；
> - 运行时，每次只有一个根签名，但是可以有多个描述符堆；
> - 描述符是一种中间层，可以给一个资源绑定多个不同类型的描述符，但要保证CPU端和GPU端的描述符是一致，也即要保证，通过同样大小的偏移进行的相对寻址能对CPU Heap和GPU Heap的结果是一致的；
> - 根签名与描述符的最终联系是在渲染时，通过设置寄存器的插槽数据对应上的。

# 7. 编译着色器
## 7.1. 离线编译
FXC 

# 8. 光栅器状态
由结构体D3D12_RASTERIZER_DESC来表示：
- FillMode:线框模式/实体模式
- CullMode
- FrontCounterClockwise：三角形绕序

# 9. 流水线状态对象 Pipeline State Object(PSO)
创建：
- D3D12_GRAPHICS_PIPELINE_STATE_DESC:
  - InputLayout: 跟顶点结构联系上了
- ID3D12Device::CreateGraphicPipelineState创建PSO对象

# 10. 问题记录
## 10.1. 内存对齐问题
```
//结构1：
struct VertexIn
{
  XMFloat4 color;
  XMFloat3 pos;
}

//输入布局1：
m_input_layout =
{
  D3D12_INPUT_ELEMENT_DESC{"POSITION", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0},
  D3D12_INPUT_ELEMENT_DESC{"COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0}
};

//结构2：
struct VertexIn
{
  XMFloat3 pos;
  XMFloat4 color;
}
//输入布局2：
m_input_layout =
{
  D3D12_INPUT_ELEMENT_DESC{"POSITION", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 16, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0},
  D3D12_INPUT_ELEMENT_DESC{"COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0}
};

//面对同一顶点数据
std::array<Vertex, 5> vertexes =
{
  Vertex({ XMFLOAT3(-1, -1, -1), XMFLOAT4(Colors::Green)}),
  Vertex({ XMFLOAT3(-1, -1, 1), XMFLOAT4(Colors::Green)}),
  Vertex({ XMFLOAT3(1, -1, 1), XMFLOAT4(Colors::Green)}),
  Vertex({ XMFLOAT3(1, -1, -1), XMFLOAT4(Colors::Green)}),
  Vertex({ XMFLOAT3(0, 5, 0), XMFLOAT4(Colors::Red)})
};
```
第一种方式读取的最后一个顶点一直为XMFLOAT3(0, 0, 0)。

推测是内存优化\内存对齐导致的。

[RootSignatureMem]: ./GPURootSignature.png


   



