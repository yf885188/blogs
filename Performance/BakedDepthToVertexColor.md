### 需求
- 网格生成并将深度信息存到网格定点色
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