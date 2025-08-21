# 排序算法

# 排序算法知识梳理

## 1. 比较排序算法

### 1.1 简单排序算法

#### 冒泡排序 (Bubble Sort)
- **原理**：重复遍历列表，比较相邻元素并交换顺序错误的元素
- **时间复杂度**：
  - 最好情况(已排序)：$O(n)$
  - 平均和最坏情况：$O(n^2)$
- **空间复杂度**：$O(1)$ (原地排序)
- **稳定性**：稳定

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        if not swapped:
            break
```

#### 选择排序 (Selection Sort)
- **原理**：每次从未排序部分选择最小(大)元素放到已排序部分的末尾
- **时间复杂度**：始终 $O(n^2)$
- **空间复杂度**：$O(1)$
- **稳定性**：不稳定

```python
def selection_sort(arr):
    for i in range(len(arr)):
        min_idx = i
        for j in range(i+1, len(arr)):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
```

#### 插入排序 (Insertion Sort)
- **原理**：构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入
- **时间复杂度**：
  - 最好情况(已排序)：$O(n)$
  - 平均和最坏情况：$O(n^2)$
- **空间复杂度**：$O(1)$
- **稳定性**：稳定

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i-1
        while j >= 0 and key < arr[j]:
            arr[j+1] = arr[j]
            j -= 1
        arr[j+1] = key
```

### 1.2 高效排序算法

#### 归并排序 (Merge Sort)
- **原理**：分治法，将数组分成两半分别排序，然后合并
- **时间复杂度**：始终 $O(n \log n)$
- **空间复杂度**：$O(n)$ (非原地排序)
- **稳定性**：稳定

```python
def merge_sort(arr):
    if len(arr) > 1:
        mid = len(arr)//2
        L = arr[:mid]
        R = arr[mid:]
        merge_sort(L)
        merge_sort(R)
        
        i = j = k = 0
        while i < len(L) and j < len(R):
            if L[i] < R[j]:
                arr[k] = L[i]
                i += 1
            else:
                arr[k] = R[j]
                j += 1
            k += 1
        while i < len(L):
            arr[k] = L[i]
            i += 1
            k += 1
        while j < len(R):
            arr[k] = R[j]
            j += 1
            k += 1
```

#### 快速排序 (Quick Sort)
- **原理**：分治法，选择一个基准元素，将数组分为小于基准和大于基准的两部分，递归排序
- **时间复杂度**：
  - 最好和平均情况：$O(n \log n)$
  - 最坏情况(已排序或逆序)：$O(n^2)$
- **空间复杂度**：$O(\log n)$ (递归栈空间)
- **稳定性**：不稳定

```python
def partition(arr, low, high):
    pivot = arr[high]
    i = low - 1
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i+1

def quick_sort(arr, low, high):
    if low < high:
        pi = partition(arr, low, high)
        quick_sort(arr, low, pi-1)
        quick_sort(arr, pi+1, high)
```

#### 堆排序 (Heap Sort)
- **原理**：利用堆数据结构，将数组构建为大顶堆，然后逐个取出堆顶元素
- **时间复杂度**：始终 $O(n \log n)$
- **空间复杂度**：$O(1)$ (原地排序)
- **稳定性**：不稳定

```python
def heapify(arr, n, i):
    largest = i
    l = 2 * i + 1
    r = 2 * i + 2
    if l < n and arr[i] < arr[l]:
        largest = l
    if r < n and arr[largest] < arr[r]:
        largest = r
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)

def heap_sort(arr):
    n = len(arr)
    for i in range(n//2 - 1, -1, -1):
        heapify(arr, n, i)
    for i in range(n-1, 0, -1):
        arr[i], arr[0] = arr[0], arr[i]
        heapify(arr, i, 0)
```

## 2. 非比较排序算法

### 计数排序 (Counting Sort)
- **原理**：统计每个元素的出现次数，然后计算每个元素在输出数组中的位置
- **时间复杂度**：$O(n + k)$，其中k是输入范围
- **空间复杂度**：$O(n + k)$
- **稳定性**：稳定
- **限制**：仅适用于整数且范围不大的情况

```python
def counting_sort(arr):
    max_val = max(arr)
    count = [0] * (max_val + 1)
    output = [0] * len(arr)
    
    for num in arr:
        count[num] += 1
    
    for i in range(1, len(count)):
        count[i] += count[i-1]
    
    for num in reversed(arr):
        output[count[num]-1] = num
        count[num] -= 1
    
    return output
```

### 桶排序 (Bucket Sort)
- **原理**：将元素分配到有限数量的桶中，每个桶再单独排序
- **时间复杂度**：
  - 最好情况：$O(n + k)$
  - 平均情况：$O(n + n^2/k + k)$
  - 最坏情况：$O(n^2)$
- **空间复杂度**：$O(n + k)$
- **稳定性**：取决于桶内排序算法的稳定性

```python
def bucket_sort(arr, bucket_size=5):
    if len(arr) == 0:
        return arr
    
    min_val = min(arr)
    max_val = max(arr)
    
    bucket_count = (max_val - min_val) // bucket_size + 1
    buckets = [[] for _ in range(bucket_count)]
    
    for num in arr:
        buckets[(num - min_val) // bucket_size].append(num)
    
    sorted_arr = []
    for bucket in buckets:
        insertion_sort(bucket)  # 可以使用其他排序算法
        sorted_arr.extend(bucket)
    
    return sorted_arr
```

### 基数排序 (Radix Sort)
- **原理**：按照低位到高位的顺序依次对数字进行排序
- **时间复杂度**：$O(d \cdot (n + k))$，其中d是数字位数，k是基数
- **空间复杂度**：$O(n + k)$
- **稳定性**：稳定
- **限制**：仅适用于整数或可以表示为整数的数据

```python
def counting_sort_for_radix(arr, exp):
    n = len(arr)
    output = [0] * n
    count = [0] * 10
    
    for num in arr:
        index = (num // exp) % 10
        count[index] += 1
    
    for i in range(1, 10):
        count[i] += count[i-1]
    
    i = n - 1
    while i >= 0:
        index = (arr[i] // exp) % 10
        output[count[index]-1] = arr[i]
        count[index] -= 1
        i -= 1
    
    for i in range(n):
        arr[i] = output[i]

def radix_sort(arr):
    max_val = max(arr)
    exp = 1
    while max_val // exp > 0:
        counting_sort_for_radix(arr, exp)
        exp *= 10
```

## 3. 排序算法比较

| 算法 | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度 | 稳定性 | 适用场景 |
|------|----------------|----------------|------------|--------|----------|
| 冒泡排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 | 小规模数据或基本有序数据 |
| 选择排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 不稳定 | 小规模数据 |
| 插入排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 | 小规模或基本有序数据 |
| 归并排序 | $O(n \log n)$ | $O(n \log n)$ | $O(n)$ | 稳定 | 大规模数据，需要稳定排序 |
| 快速排序 | $O(n \log n)$ | $O(n^2)$ | $O(\log n)$ | 不稳定 | 大规模数据，平均性能最好 |
| 堆排序 | $O(n \log n)$ | $O(n \log n)$ | $O(1)$ | 不稳定 | 大规模数据，原地排序 |
| 计数排序 | $O(n + k)$ | $O(n + k)$ | $O(n + k)$ | 稳定 | 整数且范围不大的数据 |
| 桶排序 | $O(n + k)$ | $O(n^2)$ | $O(n + k)$ | 取决于 | 均匀分布的数据 |
| 基数排序 | $O(d \cdot (n + k))$ | $O(d \cdot (n + k))$ | $O(n + k)$ | 稳定 | 整数或可表示为整数的数据 |

## 4. 排序算法的选择建议

1. **小规模数据(n ≤ 50)**：插入排序通常表现最好
2. **中等规模数据(50 < n ≤ 1000)**：快速排序通常是最佳选择
3. **大规模数据(n > 1000)**：
   - 如果内存充足：归并排序
   - 如果内存受限：堆排序
4. **特殊数据**：
   - 整数且范围有限：计数排序
   - 均匀分布的浮点数：桶排序
   - 多位数整数：基数排序
5. **需要稳定排序**：归并排序、计数排序、基数排序

## 5. 高级话题

### 5.1 混合排序算法

实际应用中常结合多种排序算法的优点：

- **内省排序(Introsort)**：结合快速排序、堆排序和插入排序
- **Timsort**：Python内置排序算法，结合归并排序和插入排序

### 5.2 并行排序算法

利用多核处理器进行并行排序：
- **并行归并排序**
- **并行快速排序**
- **Bitonic排序**：专门为并行计算设计的排序算法

### 5.3 外部排序

处理无法全部装入内存的大规模数据：
- **多路归并排序**
- **置换选择排序**
