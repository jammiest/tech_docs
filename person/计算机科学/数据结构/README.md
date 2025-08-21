# 数据结构概述

数据结构是计算机存储、组织数据的方式，它研究的是数据的逻辑结构和物理结构以及它们之间的关系。数据结构的选择会直接影响程序的效率和性能。

## 基本数据结构分类

### 1. 线性数据结构
- **数组(Array)** ：固定大小的连续内存空间
- **链表(Linked List)** ：通过指针连接的非连续存储
  - 单链表
  - 双链表
  - 循环链表
- **栈(Stack)** ：LIFO(后进先出)结构
- **队列(Queue)** ：FIFO(先进先出)结构
  - 普通队列
  - 双端队列
  - 优先队列

### 2. 非线性数据结构
- **树(Tree)**
  - 二叉树
  - 二叉搜索树
  - AVL树
  - 红黑树
  - B树/B+树
  - 堆(Heap)
- **图(Graph)**
  - 有向图/无向图
  - 加权图
  - 邻接矩阵/邻接表表示

## 时间复杂度分析

常见时间复杂度：
- $O(1)$ - 常数时间
- $O(\log n)$ - 对数时间
- $O(n)$ - 线性时间
- $O(n \log n)$ - 线性对数时间
- $O(n^2)$ - 平方时间
- $O(2^n)$ - 指数时间

## 常用操作复杂度对比

| 数据结构   | 访问     | 搜索     | 插入     | 删除     |
| ---------- | -------- | -------- | -------- | -------- |
| 数组       | O(1)     | O(n)     | O(n)     | O(n)     |
| 链表       | O(n)     | O(n)     | O(1)     | O(1)     |
| 哈希表     | O(1)     | O(1)     | O(1)     | O(1)     |
| 二叉搜索树 | O(log n) | O(log n) | O(log n) | O(log n) |
| 栈         | O(n)     | O(n)     | O(1)     | O(1)     |
| 队列       | O(n)     | O(n)     | O(1)     | O(1)     |

## 重要算法与数据结构

### 排序算法
```python
# 快速排序示例
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)
```

### 图算法
```python
# Dijkstra最短路径算法伪代码
def dijkstra(graph, start):
    distances = {vertex: float('infinity') for vertex in graph}
    distances[start] = 0
    pq = PriorityQueue()
    pq.put((0, start))
    
    while not pq.empty():
        current_distance, current_vertex = pq.get()
        if current_distance > distances[current_vertex]:
            continue
        for neighbor, weight in graph[current_vertex].items():
            distance = current_distance + weight
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                pq.put((distance, neighbor))
    return distances
```

## 实际应用场景

1. **数据库索引** ：B树/B+树
2. **内存管理** ：堆数据结构
3. **网络路由** ：图算法
4. **编译器** ：语法树、符号表
5. **操作系统** ：进程调度队列、文件系统目录树

## 高级数据结构

1. **跳表(Skip List)** ：$O(\log n)$的搜索复杂度，比平衡树实现简单
2. **布隆过滤器(Bloom Filter)** ：空间效率高的概率数据结构
3. **Trie树** ：用于字符串搜索和前缀匹配
4. **线段树(Segment Tree)** ：用于区间查询
5. **并查集(Disjoint Set)** ：处理不相交集合的合并与查询

数据结构的选择应根据具体应用场景和操作需求来决定，理解各种数据结构的特性和适用场景是软件工程师的核心能力之一。