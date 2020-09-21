# compute shader
compute shader 不能绑定Render Target，可以用画一个四方形的方式代替。

# texture sample
当texture的format不是float类型时，使用Load来采样。

# Pixel Shader
Pixel Shader中的SV_Position.z被设定为1。

# 数据对齐
GPU 内存的数据不对的时候，需要仔细检查一下CPU端结构是否是对齐的。