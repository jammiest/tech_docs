# 查找算法

# 查找算法知识梳理

## 1. 基本查找算法

### 1.1 顺序查找 (Sequential Search)
- **原理**：逐个检查列表中的每个元素
- **时间复杂度**：
  - 最好情况：$O(1)$ (第一个元素就是目标)
  - 平均和最坏情况：$O(n)$
- **空间复杂度**：$O(1)$
- **特点**：适用于无序列表

```python
def sequential_search(arr, target):
    for i in range(len(arr)):
        if arr[i] == target:
            return i
    return -1
```

### 1.2 二分查找 (Binary Search)
- **原理**：在已排序的数组中，通过比较中间元素来缩小搜索范围
- **时间复杂度**：$O(\log n)$
- **空间复杂度**：
  - 迭代实现：$O(1)$
  - 递归实现：$O(\log n)$ (递归栈空间)
- **前提条件**：数组必须是有序的

```python
def binary_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1
```

### 1.3 插值查找 (Interpolation Search)
- **原理**：基于二分查找的改进，根据目标值估计其位置
- **时间复杂度**：
  - 平均情况：$O(\log \log n)$ (均匀分布时)
  - 最坏情况：$O(n)$
- **空间复杂度**：$O(1)$
- **适用场景**：数据均匀分布的有序数组

```python
def interpolation_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high and arr[low] <= target <= arr[high]:
        pos = low + ((target - arr[low]) * (high - low)) // (arr[high] - arr[low])
        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            low = pos + 1
        else:
            high = pos - 1
    return -1
```

## 2. 树结构查找算法

### 2.1 二叉搜索树 (Binary Search Tree, BST)
- **原理**：左子树所有节点小于根节点，右子树所有节点大于根节点
- **时间复杂度**：
  - 平均情况：$O(\log n)$
  - 最坏情况(退化为链表)：$O(n)$
- **空间复杂度**：$O(n)$

```python
class BSTNode:
    def __init__(self, key):
        self.left = None
        self.right = None
        self.val = key

def bst_search(root, key):
    if root is None or root.val == key:
        return root
    if root.val < key:
        return bst_search(root.right, key)
    return bst_search(root.left, key)
```

### 2.2 平衡二叉搜索树 (AVL树)
- **原理**：自平衡二叉搜索树，保持左右子树高度差不超过1
- **时间复杂度**：始终 $O(\log n)$
- **空间复杂度**：$O(n)$

### 2.3 红黑树 (Red-Black Tree)
- **原理**：另一种自平衡二叉搜索树，通过颜色标记和旋转操作保持平衡
- **时间复杂度**：查找操作始终 $O(\log n)$
- **空间复杂度**：$O(n)$

## 3. 哈希查找

### 3.1 哈希表 (Hash Table)
- **原理**：通过哈希函数将键映射到表中的位置
- **时间复杂度**：
  - 平均情况：$O(1)$
  - 最坏情况：$O(n)$ (所有键冲突)
- **空间复杂度**：$O(n)$

```python
def hash_search(hash_table, key):
    hash_key = hash(key) % len(hash_table)
    bucket = hash_table[hash_key]
    for k, v in bucket:
        if k == key:
            return v
    return None
```

## 4. 高级查找算法

### 4.1 B树/B+树
- **原理**：多路平衡搜索树，特别适合磁盘存储系统
- **时间复杂度**：$O(\log n)$
- **空间复杂度**：$O(n)$
- **应用场景**：数据库索引、文件系统

### 4.2 跳表 (Skip List)
- **原理**：多层链表结构，通过"快速通道"加速查找
- **时间复杂度**：平均 $O(\log n)$，最坏 $O(n)$
- **空间复杂度**：$O(n \log n)$
- **特点**：实现简单，支持高效并发操作

```python
import random

class SkipListNode:
    def __init__(self, val, level):
        self.val = val
        self.next = [None] * level

class SkipList:
    def __init__(self, max_level=16, p=0.5):
        self.max_level = max_level
        self.p = p
        self.level = 1
        self.header = SkipListNode(None, max_level)
    
    def random_level(self):
        level = 1
        while random.random() < self.p and level < self.max_level:
            level += 1
        return level
    
    def search(self, target):
        current = self.header
        for i in range(self.level-1, -1, -1):
            while current.next[i] and current.next[i].val < target:
                current = current.next[i]
        current = current.next[0]
        if current and current.val == target:
            return current
        return None
```

### 4.3 布隆过滤器 (Bloom Filter)
- **原理**：概率型数据结构，用于判断元素是否可能在集合中
- **时间复杂度**：$O(k)$，k为哈希函数数量
- **空间复杂度**：$O(m)$，m为位数组大小
- **特点**：可能有假阳性，但不会有假阴性

```python
import mmh3
from bitarray import bitarray

class BloomFilter:
    def __init__(self, size, hash_count):
        self.size = size
        self.hash_count = hash_count
        self.bit_array = bitarray(size)
        self.bit_array.setall(0)
    
    def add(self, item):
        for seed in range(self.hash_count):
            index = mmh3.hash(item, seed) % self.size
            self.bit_array[index] = 1
    
    def contains(self, item):
        for seed in range(self.hash_count):
            index = mmh3.hash(item, seed) % self.size
            if not self.bit_array[index]:
                return False
        return True
```

## 5. 查找算法比较

| 算法 | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度 | 前提条件 | 适用场景 |
|------|----------------|----------------|------------|----------|----------|
| 顺序查找 | $O(n)$ | $O(n)$ | $O(1)$ | 无 | 无序小规模数据 |
| 二分查找 | $O(\log n)$ | $O(\log n)$ | $O(1)$ | 有序数组 | 静态有序数据 |
| 插值查找 | $O(\log \log n)$ | $O(n)$ | $O(1)$ | 有序且均匀分布 | 均匀分布的大规模数据 |
| 二叉搜索树 | $O(\log n)$ | $O(n)$ | $O(n)$ | 二叉搜索树结构 | 动态数据集 |
| AVL树 | $O(\log n)$ | $O(\log n)$ | $O(n)$ | 平衡二叉搜索树 | 需要严格平衡的场景 |
| 红黑树 | $O(\log n)$ | $O(\log n)$ | $O(n)$ | 红黑树结构 | 需要高效插入删除的场景 |
| 哈希查找 | $O(1)$ | $O(n)$ | $O(n)$ | 良好的哈希函数 | 快速查找，不要求有序 |
| B树/B+树 | $O(\log n)$ | $O(\log n)$ | $O(n)$ | 多路平衡树 | 磁盘存储系统 |
| 跳表 | $O(\log n)$ | $O(n)$ | $O(n \log n)$ | 多层链表结构 | 需要简单实现的并发场景 |
| 布隆过滤器 | $O(k)$ | $O(k)$ | $O(m)$ | 多个哈希函数 | 集合成员概率判断 |

## 6. 查找算法的选择建议

1. **静态数据集**：
   - 如果数据已排序：二分查找或插值查找
   - 如果数据无序且规模小：顺序查找

2. **动态数据集**：
   - 需要高效查找和更新：平衡二叉搜索树(AVL树、红黑树)
   - 需要最快查找：哈希表(如果不需有序遍历)

3. **大规模外部存储**：
   - B树/B+树(数据库索引)
   - LSM树(日志结构合并树，如LevelDB)

4. **特殊需求**：
   - 需要空间效率：布隆过滤器(判断元素是否存在)
   - 需要并发操作：跳表
   - 需要范围查询：B+树

## 7. 高级话题

### 7.1 近似查找算法
- **局部敏感哈希(LSH)**：用于相似项查找
- **最近邻搜索(NNS)**：在高维空间中查找最近邻

### 7.2 分布式查找
- **一致性哈希**：用于分布式系统中的数据分布和查找
- **分布式哈希表(DHT)**：如Chord、Pastry等算法

### 7.3 量子查找算法
- **Grover算法**：量子计算中的无序数据库搜索算法，时间复杂度为$O(\sqrt{n})$




