
![][Pipeline]

[Pipeline]: ./images/Pipeline.jpg

# vs

![][VertexShader]

[VertexShader]: ./images/VertexShader.jpg

## 内建
- 内建变量
- 内建统一状态
  - glDepthRange: 包括near，far和diff(far - near)
- 内建常量

## 精度限定符

## VS中的uniform限制数量
- uniform和const 变量、字面值等必须按照**打包规则**之后与gl_MaxVertexUniformVectors相匹配
- 字面值最好能统一，因为字面值在shader内会多次计算

# 变换反馈
直接把VS的输出到缓冲区对象中。
- glTransformFeedbackVaryings:捕捉对应变量并输出到缓冲区
- glBeginTransformFeedback
- glEndTransformFeedback