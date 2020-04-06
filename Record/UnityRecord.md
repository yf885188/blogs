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