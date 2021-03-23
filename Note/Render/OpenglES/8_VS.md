
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