### 需求
- 网格生成并将深度信息存到网格定点色
- 网格合并
- 有对应的编辑界面
- 把整个界面打包成一个package

### 准备及注意事项
- UI使用的Odin插件，使用当前插件之前确保先安装了Odin
- 深度相机虽然使用的是正交投影，但是需要保证深度在所有的场景上方保证不会有面片穿过相机的投影导致，场景面片背后的物体深度覆盖了真实深度
- 因为在urp中，实际使用的是材质替代渲染的类似方式，需要设置layermask保证哪些场景需要渲染，哪些不用 

### 实现
#### 网格生成并将深度信息存到网格定点色
#### 生成深度信息
采用材质替代渲染的方式，设置相机为正交投影，设置Rect刚好为平面的大小，在shader中计算世界坐标，跟设置的高度范围进行对比，得到深度信息。
```
//相机的初始设置
void InitCamera(GameObject plane)
{
    //根据plane设置相机相关参数
    m_cam.targetTexture = m_rt;
    m_cam.orthographic = true;
    m_cam.nearClipPlane = 0.1f;
    m_cam.farClipPlane = 1000f;
    m_cam.transform.forward = new Vector3(0, -1, 0);
    float width = m_param.Size.x * m_param.UnitSize;
    float height = m_param.Size.y * m_param.UnitSize;
    m_cam.aspect = height / width;
    m_cam.enabled = true;
    m_cam.orthographicSize = width;
    m_cam.clearFlags = CameraClearFlags.SolidColor;
    m_cam.backgroundColor = Color.black;
    m_cam.gameObject.transform.position = plane.transform.position + new Vector3(0, 1, 0);
}
```

```
//材质替代渲染
void RenderDepth()
{
    Shader.SetGlobalFloat("_DepthRangeBottom", m_param.Bottom);
    Shader.SetGlobalFloat("_DepthRangeTop", m_param.Top);
    //RenderWithShader在URP中被弃用了，所以下面的做法不能达到效果
    //m_cam.RenderWithShader(m_param.DepthShader, "RenderType");
    //使用下面的方式进行替代
    ScriptableObject obj = AssetDatabase.LoadAssetAtPath(@"Assets/Settings/UniversalRP-LowQuality.asset", typeof(ScriptableObject)) as ScriptableObject;
    SerializedObject se_obj = new SerializedObject(obj);
    SerializedProperty pro = se_obj.FindProperty("m_DefaultRendererIndex");
    pro.intValue = 1;
    se_obj.ApplyModifiedProperties();
    m_cam.Render();
}
```
>这里需要注意的是，因为采用URP的关系，原有的Camera.RenderWithShader()的方式不能达到目的。在不破坏URP源代码的情况下，这里采用更改RendererData的方式，使用自带的RenderObjectsPass中的overrideMaterial来实现材质替代渲染。

### 合并网格
处理海面太大的情况，把海面plane给分割成好几个patch，分批对mesh的定点色进行赋值：移动深度相机对patch生成深度图，并对深度图进行采样赋值给网格顶点色。
```
//使用unity自带的网格合并，不会合并相同定点，不再划分sub mesh
FillParam();
//分割mesh信息
Mesh mesh = m_gen_mesh_util.GenMesh(new Vector2(m_param.UnitSize, m_param.UnitSize), OceanObj);
List<PatchInfo> sub_patch_info = m_gen_mesh_util.DividePatch(OceanObj, m_param, mesh);

m_combine_instances.Clear();
Color[] cols = new Color[mesh.vertexCount];
for(int i=0; i<sub_patch_info.Count; ++i)
{
    
    //根据mesh信息烘焙
    RenderTexture rt = m_bake_depth_util.Execute(sub_patch_info[i].MeshPos + new Vector3(0, SceneMaxHeight, 0));
    SaveDepthTexture(rt);
    //更新定点色
    m_gen_mesh_util.MapRTToVertexColor(rt, sub_patch_info[i], in m_param, mesh, cols);
}
SetColorerMesh(mesh, cols);
```

### 网格优化
主要做的优化：
1、 在水平面以下的面片去掉
2、 非海边的面片合并成大的面片

主要实现的算法：
进行四叉树合并；
原始的节点均为size最小的叶子节点；
海边的面片设置成dirty，不可进行合并；
高于海面一定距离的面片被剔除；
其他的面片节点可以按照四叉树进行合并成大的面片；

合并的具体做法：
0、设置一个记录当前定点取下一个定点的step值的容器，如果step大于1，表示当前的定点出于一个已经合并的quad中，直接加step到下一个定点，否则跳到1；
1、判断当前的quad中的四个点定点色是否都是在判断范围内，是的话，找下一个点，不是的话，转到2；
2、查找相邻的quad，判断quad是否能合并，如果能则继续这一条，不能的话转到3；
3、更新记录step的容器，然后调到0。

等所有格子合并完成之后，对所有的顶点进行压缩挪位。
