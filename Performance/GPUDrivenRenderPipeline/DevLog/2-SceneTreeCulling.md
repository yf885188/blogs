# 目标
- 使用四叉树
- 存成离线文件

# 思路
- 生成四叉树
  - 根据AABB来确定场景大小
  - 从上至下来分割四叉树的节点
- 深度遍历存储离线文件

## 生成四叉树
参考[网址](https://zhuanlan.zhihu.com/p/180560098)

- 按照场景尺寸和预设的场景树最大深度划分层次
- 节点插入到层次中：直接根据大小和稀疏四叉树的关系，单个插入的复杂度为$O(1)$
- 从上往下遍历这个层次，得到最终的场景树
  - 高层的节点中包含AABB，说明这个节点划分的后续节点要合并到这个高层节点中

## 提交数据
在场景开始把所有顶点都提交到GPU，然后通过场景树进行剔除然后选择绘制。

> 不根据剔除后的Mesh进行提交是因为：
> - 每帧的顶点数量都不一样，这样的话，每帧创建资源映射太消耗时间

## 结果对比
<div align="center">

![][SceneTreeDS]

延迟渲染的结果

![][SceneTreeZB]
ZBuffer渲染的结果

</div>

[SceneTreeDS]: ./SceneTreeCullingDS.jpg
[SceneTreeZB]: ./SceneTreeCullingZB.jpg
