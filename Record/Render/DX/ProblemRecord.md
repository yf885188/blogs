# compute shader
compute shader 不能绑定Render Target，可以用画一个四方形的方式代替。

# texture sample
当texture的format不是float类型时，使用Load来采样。

# Pixel Shader
Pixel Shader中的SV_Position.z被设定为1。

# 数据对齐
GPU 内存的数据不对问题/数据访问导致的崩溃问题，需要仔细检查一下CPU端结构是否是对齐的/CPU端与GPU端的结构是否对齐。
UAV视图要4096对齐。Cbuffer对应的结构要256对齐。