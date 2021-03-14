# 渲染的基本对象
- 着色器对象
- 程序对象：类似链接程序。

# shader
- 创建Shader句柄
- 绑定着色器源码
- 编译

> 容错机制：
>   - glGetShaderiv
>   - glGetShaderInfoLog

# program
- 创建
- 绑定shader：在绑定的时候，shader可以没有编译，没有源代码。但是必须要有一个vs和一个fs绑定。
- 链接program
  - 检查各类对象的数量
  - 检查vs的输出和fs的输入一致
  - 检查属性/统一变量/输入输出着色器变量的数量
- 使用：glUseProgram

> 容错机制：
>   - glGetProgramiv
>   - glGetProgramInfoLog
>   - glValidateProgram

# uniform
## 分类
- 命名统一变量块
- 默认的统一变量块

## 获取和设置
- 获取：
  - glGetProgramiv：获取状态
  - glGetActiveUniform
  - glGetActvieUnifromsiv
  - glGetUniformLocation
- 设置：
  - glUniformXY(GLint location, GLType x)等类型
  
# 统一变量缓冲区对象 uniform buffer
  - 可以用来在shader/program之间共享数据
  - 降低API开销
  - 数据对齐问题
  - 矩阵的数据排列问题
  - uniform布局：layout
  - 使用：
    - 获取：
    - 绑定：glUniformBlockBinding/glBindBufferRange/glBindBufferBase
  - 限制：
    - 单shader的数量限制
    - 当program的数量限制
    - 单buffer的大小限制：最小的最大支持尺寸为16KB

# 属性

# shader和program的编译和预编译