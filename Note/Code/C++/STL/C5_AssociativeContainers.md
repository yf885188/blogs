# 总体
- 关联式容器的内部结构是一个balanced binary tree（平衡二叉树）

# tree
- 二叉搜索树
- 平衡二叉搜索树
- AVL tree

## AVL Tree
任何节点的左右子树高度相差最多1。

### 不平衡之后的旋转操作
分类：
- 外侧插入：采用单旋转操作
  - 左左
  - 右右
- 内侧插入：双旋转操作
  - 左右
  - 右左

只要调整“插入点至根节点”路径上，平衡状态被破坏之各节点中最深的那一个，便可使整棵树重新获得平衡。

#### 单旋转
![][SingleRotation]

[SingleRotation]: ./SingleRotation.jpg

#### 双旋转
两次单旋转。

## RB-Tree
比AVL Tree的平衡性要弱，不保证左右子树的深度只差1。

特殊的二叉搜索树，满足以下规则：
- 每个节点不是红色就是黑色
- 根节点为黑色
- 如果节点为宏，那么子节点必定为黑
- 任意节点至NULL的任何路径，所含之黑节点书必须相同

![][RBTreeStructure]

[RBTreeStructure]: ./RBTreeStructure.jpg

### 旋转实现
![][RBTreeRotation0]

[RBTreeRotation0]: ./RBTreeRotation0.jpg

![][RBTreeRotation1]

[RBTreeRotation1]: ./RBTreeRotation1.jpg

> - 深度的不平衡控制了是否旋转
> - 规则3和规则4控制了是否变色

### 节点设计
节点双层结构。

> 为了防止边界情况的发生，专门设置了为根节点设计了一个父节点——header节点。

# set
- set的iterator是RB-Tree的const_iterator，是一种 constant iterator

# map

# multiset
与set一样，不过允许value重复。

# multimap
与map一样，不过允许key重复。

# hashtable
## hash function（散列函数）
- 碰撞问题：
  - 线性探测：主集团问题
  - 二次探测：次集团问题——复式散列
  - 开链（separate chaining）



### 开链——桶子（buckets）与节点（nodes）

![][HashTable]

[HashTable]: ./HashTable.jpg

是否要resize重建表格，完全看buckets vector是否要取下一个size的质数。

# hash_set
一般的set以RB-Tree作为底层机制，而hash_set以hashtable作为底层机制。

# hash_map

# hash_multiset

# hash_multimap

