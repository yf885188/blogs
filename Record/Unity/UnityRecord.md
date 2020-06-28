### RenderTexture相关
#### RenderTexture 保存成PNG
```
static public void SaveRenderTextureToPNG(RenderTexture rt, string path)
{
    //io限制
    float delta_time = Time.time - m_last_time;
    if(delta_time < 0.01f)
    {
        //return;
    }
    m_last_time = Time.time;
    
    RenderTexture cur_rt = RenderTexture.active;
    RenderTexture.active = rt;
    Texture2D tex = new Texture2D(rt.width, rt.height);
    tex.ReadPixels(new Rect(0, 0, tex.width, tex.height), 0, 0);
    byte[] pixels = tex.EncodeToPNG();
    string folder_path = path.Substring(0, path.LastIndexOf(@"/"));
    if(!Directory.Exists(folder_path))
    {
        Directory.CreateDirectory(folder_path);
    }
    FileStream stream = File.Open(path, FileMode.OpenOrCreate);
    BinaryWriter writer = new BinaryWriter(stream);
    writer.Write(pixels);
    stream.Close();
    Texture2D.DestroyImmediate(tex);
    tex = null;
    RenderTexture.active = cur_rt;
}
```

#### RenderTexture 保存成Asset
RenderTexutre 作为一个GPU上分配的一块显存，想要持久化保存的话，最好保存成图片，不要通过asset的方式进行持续化保存。

#### Shader 中的loop展开
Shader中写loop的时候需要根据变量设置循环的次数，在有些平台上无法正常展开，或者不支持dynamic loop。可以尝试一下方式：
- Opengl转到Dx11及以上平台
- 在shader中添加平台相关的控制：
```
#pragma exclude_renderers d3d11_9x
#pragma target 4.0
```

#### URP Additional Light Shadow 只支持聚光灯光源阴影

<div align="center">

![][AdditionalShadowOnlySpot]

</div>

#### 修改材质属性

如果通过get material的方式先获取再复制，会先将material实例化。并且在有些情况下会出现bug，比如在ios中实际上给transform添加了多材质等。

应该使用MaterialPropertyBlock的方式，使用封装的方式来更新材质属性。


#### 平面在烘焙lightmap和ShadowMask的时候背面效果不对的问题
推测是因为平面没有厚度造成的，减少平面的此类应用。

[AdditionalShadowOnlySpot]: ./AddtionalShadowOnlySpot.png