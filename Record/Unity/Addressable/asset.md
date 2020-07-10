<!-- TOC -->

    - [File GUIDs、Local IDs和Instance ID](#file-guidslocal-ids和instance-id)
    - [Object 生命周期](#object-生命周期)
    - [特殊目录](#特殊目录)
    - [资源加载方式](#资源加载方式)
- [Asset Bundle](#asset-bundle)
    - [结构](#结构)
        - [包头](#包头)
            - [查找表](#查找表)
            - [压缩方式](#压缩方式)
    - [加载AssetBundle方式](#加载assetbundle方式)
    - [从AssetBundle加载Assets](#从assetbundle加载assets)
    - [加载依赖](#加载依赖)
    - [加载的细节处理](#加载的细节处理)
    - [发布](#发布)
        - [下载](#下载)
        - [存储](#存储)
    - [新资源导入管线](#新资源导入管线)
    - [Unity Accelerator](#unity-accelerator)
- [Addressable](#addressable)
    - [相关概念](#相关概念)
    - [加载特殊处理](#加载特殊处理)
    - [AddressabeReference](#addressabereference)
    - [Asset管理](#asset管理)
        - [Asset Group Schemas](#asset-group-schemas)
    - [Asset更新工作流](#asset更新工作流)
    - [托管服务设置](#托管服务设置)
    - [内存管理](#内存管理)
    - [分析器](#分析器)

<!-- /TOC -->

## File GUIDs、Local IDs和Instance ID
- File GUID : Asset 在全局中的ID
- Local ID : Object 在Asset中的位置
- Instance ID : 运行时id，具体参看[PersistentManager](https://edu.uwa4d.com/course-intro/0/113?)

## Object 生命周期
- 加载
- 卸载

## 特殊目录
- Editor
- EditorDefaultResources
- Plugins
- Resources ： 同步加载。对比AssetBundles使用异步加载的防止。
- Gizmos
- StreamingAssets
> 资源打包出去的唯二文件夹： Resources 和 StreamingAssets

## 资源加载方式
- Resources ： 资源全部加载，常驻内存
- AssetBundles ： 包内资源不可见；回调复杂；调试复杂导致的开发不便等。
- AssetDatabase ： 同步加载

# Asset Bundle
提供一种压缩文件的格式，把1个/多个文件进行索引和序列化。在运行时，根据不同情况加载优化后的资源。

## 结构
- 包头：
- 数据段

### 包头
#### 查找表
使用查找表记录信息
- key 为Object Name。
- 底层实现为平衡搜索树（红黑树等）

#### 压缩方式
- LZMA ： 生成后的整个数据包进行压缩
- LZ4 ： Asset可以单个压缩

## 加载AssetBundle方式
- LoadFromMemory : 资源存在多份冗余
- LoadFromFile ： 只针对未压缩和LZ4压缩格式
- UnityWebRequest's DownloadHandlerAssetBundle
- WWW.LoadFromCacheOrDownload

## 从AssetBundle加载Assets
- LoadAsset
- LoadAllAsset
- LoadAssetWithSubAssets

## 加载依赖
- AssetBundle的加载无先后顺序，但是Asset的加载有。

## 加载的细节处理
这篇[blog](https://zhuanlan.zhihu.com/p/98081170)写得很具体了。
- AssetBundle.Unload() : 是否会造成资源冗余等问题。 

## 发布
- 随项目发布： streaming asset
- 安装后下载：

### 下载

### 存储
- Application.PersistentDataPath : 存储在应用程序之间需要持久化的数据，可写。
- Application.streamingAssetPath : 不可写。

## 新资源导入管线
- 导入结果：资源导入到Unity之后会使用一个内部格式
- 定论： 平台无关
- 依赖追踪： 依赖的资源发生变化会导致整个依赖链上的Asset都过时

## Unity Accelerator
一个用来加速资源管理的协作服务。存储的是已经转换后的Asset，减少了asset的格式转换时间，实现加速。

# Addressable
主要优势：把对Asset的规划、构建和加载进行解耦。

<div align="center">

![Addressable优势][AddressableAdvantages]

</div>

## 相关概念
<div align="center">

![Addressable专有名词][AddressableConcepts]

</div>

## 加载特殊处理
- 加载components
- 加载 sub-assets

## AddressabeReference

## Asset管理
### Asset Group Schemas

## Asset更新工作流
- 将资源分类（Static和Dynamic），并进行状态设置
- 检查Asset修改的更新
- 构建Asset更新：根据addressable_content_state.bin来创建asset bundles。
> Dynamic AssetBundle都是按组一起更新的，Static AssetBundle则可以单独更新Assetbundle

## 托管服务设置

## 内存管理
详细的可以看这篇[blog](https://zhuanlan.zhihu.com/p/98663058)。
> 需要注意： asset引用计数归零了，不一定资源被卸载了，跟asset bundle的卸载时机以及自己的卸载方式有关。

## 分析器

[AddressableConcepts]: ./AddressableConcepts.jpg
[AddressableAdvantages]: ./AddressableAdvantages.jpg