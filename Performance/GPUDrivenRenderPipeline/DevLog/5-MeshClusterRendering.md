# 目标
- 每帧提交顶点数据，绑定为Object数据
- 根据Hi-Z进行Object数据的剔除
- 生成Mesh Cluster
- 对Mesh Cluster进行剔除
- Index Buffer压缩

# 思路
## 顶点数据的提交
以前的顶点提交都是静态提交：
- 在进入场景的时候把所有的顶点数据压入，在场景树剔除后，从静态的顶点数据中取数据绘制
- 为了方便定位数据，都是使用的const buffer存数据

主要针对上面两点进行更改。
- 每帧提交顶点数据话，需要注意，使用ring buffer 而不是以前的帧资源进行处理，需要顶点和索引buffer都是连续的。之前的处理把所有的数据都当做是chunk进行处理，不能进行类似处理的mat单独建了一个buffer。
- 顶点等各项数据提交之后，不再read back回CPU。在GPU端各阶段的处理，比如：hi-z进行的Instance culling，使用追加缓冲区/消费缓冲区等方式进行处理。

## culling
包括两种：instance culling和cluster culling

### instance culling
具体思路：
- 在Object Buffer中加入包围盒信息。


# 实践
很多结果都是在GPU上产生的，如果使用传统方式，会产生从GPU到CPU的回传，这里使用，ExecuteIndirect的方式进行处理。具体可以参看官方的例子D3D12ExecuteIndirect。