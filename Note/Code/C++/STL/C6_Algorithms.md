# 概述
质变算法：
- 质变算法 mutating algorithms：会改变操作对象的值
  - 通常提供两个版本：in-place就地版本；copy另地版本
- 非质变算法 nonmutating algorithms

仿函数：
- 缺省算法
- 仿函数

# 算法的泛化
总体思路：把操作对象的型别加以抽象化，把操作对象的标示法和区间目标的移动行为抽象化，整个算法就会工作在一个抽象层面上。

# 数值算法
头文件<numeric>

# 基本算法
![][AlgorithmsCopy]

[AlgorithmsCopy]: ./AlgorithmsCopy.jpg

# set相关算法
- 并集
- 交集
- 差集
- 对称差集

> 必须是sorted

# heap算法

# 其他算法
- next_permutation与prev_permutation

## sort排序
- 数据量大时采用Quick Sort，分段递归排序
- 分段数据量小了之后会只用插入排序
- 如果递归层次过审，改用堆排序

### intro sort -- median of three quick sort
- 必须是RandomAccessIterator