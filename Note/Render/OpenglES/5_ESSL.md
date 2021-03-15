# 细节
- 函数不能递归
- 在编译时已知值的变量应该是常量，而不是统一变量，可以提高效率
- uniform的数量有限制
- uniform的布局可以全局默认声明，也可以被局部覆盖
- uniform通常保存在“常量存储”中
- VS/FS的输入/输出属性也有数量限制
- VS的输出和FS的输入不需要布局，但是VS的输入可以有布局，FS的输出由于有MRT的存在，也是可以有布局的
- 插值限定符
  - smooth
  - 平面着色 flat：以某一个顶点的值作为平面的值
- 预处理指令
  - #error
  - #pragma
  - #extension
- 基于打包规则，不能使用打包之后超过最小运行存储大小的uniform或者插值器
- invariant会有性能问题，需要注意

# 打包规则
物理存储空间被组织成vec4的形式。

![][StoragePacking]

[StoragePacking]: ./images/StoragePacking.jpg

# 精度限定符
- lowp
- mediump
- highp

# invariant
规定统一规则的计算，输入相同，输出也要相同。