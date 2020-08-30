# compute shader
compute shader 不能绑定Render Target，可以用画一个四方形的方式代替。

# texture sample
当texture的format不是float类型时，使用Load来采样。