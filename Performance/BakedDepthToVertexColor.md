### 需求
- 网格生成并将深度信息存到网格定点色
- 网格合并
- 有对应的编辑界面
- 把整个界面打包成一个package

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
处理海面太大的情况，把海面plane给分割成好几个patch，分批进行网格生成，然后移动深度相机对patch生成深度图，并对深度图进行采样赋值给网格顶点色。
```
List<MeshInfo> sub_mesh_info = m_gen_mesh_util.DivideSubMesh(OceanObj, m_param);
Mesh combined_mesh = new Mesh();
combined_mesh.name = "CombinedMesh";
List<CombineInstance> combine_instances = new List<CombineInstance>();
for(int i=0; i<sub_mesh_info.Count; ++i)
{
    
    //根据mesh信息烘焙
    RenderTexture rt = m_bake_depth_util.Execute(sub_mesh_info[i].MeshPos);
    //更新定点色
    m_gen_mesh_util.MapRTToVertexColor(rt, sub_mesh_info[i], in m_param);

    //把更新好的mesh全都合并成一个新的mesh，并赋值给原plane
    CombineInstance combine_instance = new CombineInstance();
    combine_instance.mesh = sub_mesh_info[i].Mesh;
    combine_instance.transform = sub_mesh_info[i].Transform.localToWorldMatrix;
    
    combine_instances.Add(combine_instance);
}
Debug.Log(" combine sub mesh count : " + combine_instances.Count.ToString());
combined_mesh.CombineMeshes(combine_instances.ToArray());
OceanObj.GetComponent<MeshFilter>().mesh = combined_mesh;
//销毁掉sub mesh 的gameobject
foreach(var meshinfo in sub_mesh_info)
{
    GameObject.DestroyImmediate(meshinfo.Transform.gameObject);
}
```
