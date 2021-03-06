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
- 把包围盒投影到ndc空间
- 根据ndc空间的包围盒大小，选较长边确定Hi-Z的层级，保证至少能覆盖两个纹素
- 根据ndc空间包围盒4点采样的Hi-Z上的深度，跟最近深度（这里简单处理，取8个顶点中的最近深度）进行比较。
  - 如果4个采样深度中，存在一个采样深度大于最近深度，则intance不被剔除
  - 反之，则被剔除

### chunk expan
- Instance Culling 完毕之后单个实体的顶点数差异可能会很大，可以先粗分为几个chunk，chunk里包含多个cluster。
- cluster Bounds的填充处理也放在这个pass中进行。
  - 对于面片数不是cluster面片数（这里是64）倍数的instance数据需要填充顶点
  - 因为后续的vertex/index buffer都是所有的instance公用，instance的index需要重新计算在合并后的index buffer中的index数据。放在CPU端处理的话，需要遍历。因为DrawArgument中也记录了VertexStartLocation,相当于记载了instance前的总顶点偏移量，因此可以使用index + VertexStartLocation的方式更新得到新的index。
- 使用ExecuteIndirect的时候不会记录Index数量，但是通过面片填充之后，最后绘制的是cluster，面片数量固定（这里是64）。

### cluster culling
计算跟Instance Culling类似。

# 实践
- 很多结果都是在GPU上产生的，如果使用传统方式，会产生从GPU到CPU的回传，这里使用，ExecuteIndirect的方式进行处理。具体可以参看官方的例子D3D12ExecuteIndirect。
- 复用FrameResource中的资源遇到一些问题： 
  - 因为是使用的ring buffer，所以不能像其他的资源那样直接在**初始化阶段**就指定好Object buffer的位置，动态创建描述符的性能消耗这里存疑，暂时没看出有啥问题
  - ring buffer中的对齐是按照8bits也即1B来对齐的，但是在创建视图的时候因为没有单独创建新的资源，绑定到FrameResource的时候采用偏移的方式定位，但是这种既有方式是根据视图对应的数据结构为偏移的最小元素，所以在ring buffer对各buffer加入了按照各自数据结构大小的偏移计算。
- 要仔细检查CPU跟GPU端结构的对齐情况，不然容易出现数据传输错误。tips：可以在RenderDoc中看Buffer的格式，来进行对应的修改。
- UAV视图要4096对齐。Cbuffer对应的结构要256对齐。


<div align="center">

![][RenderDocStrutureDebug]

</div>


# 结果
cluster渲染表现
<div>

![][ClusterCullingRes0]

![][ClusterCullingRes1]

</div>

最后总结的结果比原来什么都不做的处理帧率要提高50%~100%。
但还是存在一些实体pop的现象。需要后续接着改进。

[RenderDocStrutureDebug]: ./RenderDocStrutureDebug.jpg
[ClusterCullingRes0]: ./ClusterCullingRes0.jpg
[ClusterCullingRes1]: ./ClusterCullingRes1.jpg
# 待解决问题
