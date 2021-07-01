# 仿射变换 affine transform

- 定义：线性变换 + translation
- 包含：所有的translation,ratation,scaling,reflection和shearing matrices（剪切矩阵）
- 特性：保证了行的并行度

# 正交矩阵 Orthogonal matrices
- 其逆矩阵 = 其转置矩阵

# 基本变换
## Translation
- 特性： $T^{-1}(t) = T(-t)$

## Rotation
- 秩 : $tr(R) = 1 + 2\cos\phi$
- 行列式(determinant)的值为1
- 正交矩阵
- 同样有$R^{-1}_i{\phi}=R_i(-\phi)$

## Scaling
- uniform对应isotropic，nonuniform对应anisotropic
- $S^{-1}(s) = S(\frac {1} {s_x}, \frac {1} {s_y}, \frac {1} {s_z})$

> 反射矩阵/镜子矩阵：
> - 定义：S矩阵中的一个或者其他三个是负数
> - 判断：S矩阵中的左上的3x3矩阵对应的行列式的值为负数，则说明矩阵是反射矩阵。

## Shearing
- 特性： $H^{-1}_{ij}(s) = H_{ij}(-s)$

## 变换组合
- 跟顺序有关

## 刚体变换
- 定义：只考虑translation和rotation组合的变换
- $X = T(t)R$
- $X^{-1} = R^TT(-t)$

## normal transform
- 不能用点的变换来变换normal，因为要考虑scale。但是可以变换tangent。
- 使用伴随矩阵的转置来做
  - 但是 伴随矩阵 = 逆矩阵 / 行列式的值 ，行列式的值可能为0
  - 4x4计算太昂贵，很多情况下的w分量默认为1，所以通常计算3x3的伴随矩阵
  - 如果scale是uniform的，则可以直接用TRS变换normal。而其中T和S实际上对normal并没有意义。而又有$R^T = R^{-1}$

## 逆矩阵计算
- 组合：$M = T(t)R(\phi)$,则有$M^{-1} = R(-\phi)T(-t)$
- 正交矩阵： $M^{-1} = M^T$
- 其他情况，就用伴随矩阵等方式求解