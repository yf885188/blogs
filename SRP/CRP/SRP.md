- [基本步骤](#基本步骤)
- [基本结构](#基本结构)

# 基本步骤
1. 创建RP Asset，并设置当前的渲染管线为新建的RP Asset
2. 创建RP
3. 正式绘制
    1. 相机绘制loop
    2. 绘制前准备。Clear - Layers - Cull 
    3. 设置绘制顺序。(大致顺序)skybox - opaque - transparent
    
# 基本结构
1. 着色器Id **ShaderTagId** 对应的是Tags中的"LightMode"
2. 排序 **SortingSettings**
3. 绘制 **DrawingSettings**
4. 剔除 **FilteringSettings**


