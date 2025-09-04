### LeetCode 题型模式架构解析

LeetCode 题目存在可识别的模式和解法范式，掌握这些模式可以大幅提高解题效率。以下系统解析常见题型模式、识别特征和解题模板。

#### 1. 滑动窗口模式

##### 1.1 固定大小窗口
**特征**：子数组/子字符串定长问题
**模板**：
```python
def fixed_window(nums, k):
    window_sum = sum(nums[:k])
    max_sum = window_sum
    for i in range(k, len(nums)):
        window_sum += nums[i] - nums[i-k]
        max_sum = max(max_sum, window_sum)
    return max_sum
```
**例题**：643. 子数组最大平均数 I

##### 1.2 可变大小窗口
**特征**：满足条件的最短/最长子数组
**模板**：
```python
def variable_window(s, t):
    left = 0
    freq = Counter(t)
    required = len(freq)
    formed = 0
    for right in range(len(s)):
        # 更新右指针
        if formed == required:
            # 更新答案并移动左指针
    return result
```
**例题**：76. 最小覆盖子串

#### 2. 双指针模式

##### 2.1 相向指针
**特征**：排序数组，两数/三数之和
**模板**：
```python
def two_pointers(nums, target):
    left, right = 0, len(nums)-1
    while left < right:
        total = nums[left] + nums[right]
        if total == target:
            return [left, right]
        elif total < target:
            left += 1
        else:
            right -= 1
```
**例题**：167. 两数之和 II

##### 2.2 快慢指针
**特征**：链表循环检测，数组去重
**模板**：
```python
def fast_slow(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True  # 有环
    return False
```
**例题**：141. 环形链表

#### 3. 区间合并模式

##### 3.1 区间处理
**特征**：重叠区间合并，插入新区间
**模板**：
```python
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = []
    for interval in intervals:
        if not merged or merged[-1][1] < interval[0]:
            merged.append(interval)
        else:
            merged[-1][1] = max(merged[-1][1], interval[1])
    return merged
```
**例题**：56. 合并区间

#### 4.  cyclic sort模式

##### 4.1 原地排序
**特征**：数组元素在 [1, n] 范围内
**模板**：
```python
def cyclic_sort(nums):
    i = 0
    while i < len(nums):
        j = nums[i] - 1
        if nums[i] != nums[j]:
            nums[i], nums[j] = nums[j], nums[i]
        else:
            i += 1
```
**例题**：268. 丢失的数字

#### 5. 链表反转模式

##### 5.1 基本反转
**模板**：
```python
def reverse_list(head):
    prev = None
    curr = head
    while curr:
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
    return prev
```
**例题**：206. 反转链表

##### 5.2 K组反转
**模板**：
```python
def reverse_k_group(head, k):
    # 统计长度，分组反转
    # 递归或迭代实现
```
**例题**：25. K 个一组翻转链表

#### 6. 树形DFS模式

##### 6.1 路径和问题
**模板**：
```python
def path_sum(root, target):
    def dfs(node, current_sum):
        if not node:
            return False
        current_sum += node.val
        if not node.left and not node.right:
            return current_sum == target
        return dfs(node.left, current_sum) or dfs(node.right, current_sum)
    return dfs(root, 0)
```
**例题**：112. 路径总和

##### 6.2 序列化/反序列化
**模板**：
```python
def serialize(root):
    if not root:
        return "None,"
    return str(root.val) + "," + serialize(root.left) + serialize(root.right)
```
**例题**：297. 二叉树的序列化与反序列化

#### 7. 子集模式

##### 7.1 回溯法
**特征**：所有子集/组合/排列
**模板**：
```python
def subsets(nums):
    result = []
    def backtrack(start, path):
        result.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i+1, path)
            path.pop()
    backtrack(0, [])
    return result
```
**例题**：78. 子集

#### 8. 二分搜索变种

##### 8.1 旋转数组搜索
**模板**：
```python
def search_rotated(nums, target):
    left, right = 0, len(nums)-1
    while left <= right:
        mid = (left+right)//2
        if nums[mid] == target:
            return mid
        # 判断哪边有序
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid-1
            else:
                left = mid+1
        else:
            if nums[mid] < target <= nums[right]:
                left = mid+1
            else:
                right = mid-1
    return -1
```
**例题**：33. 搜索旋转排序数组

#### 9. 堆模式

##### 9.1 Top K问题
**模板**：
```python
def top_k(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap
```
**例题**：215. 数组中的第K个最大元素

##### 9.2 频率统计
**模板**：
```python
def top_k_frequent(nums, k):
    freq = Counter(nums)
    heap = []
    for num, count in freq.items():
        heapq.heappush(heap, (count, num))
        if len(heap) > k:
            heapq.heappop(heap)
    return [num for count, num in heap]
```
**例题**：347. 前 K 个高频元素

#### 10. 动态规划模式

##### 10.1 背包问题
**模板**：
```python
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0]*(capacity+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for w in range(1, capacity+1):
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i-1][w], values[i-1] + dp[i-1][w-weights[i-1]])
            else:
                dp[i][w] = dp[i-1][w]
    return dp[n][capacity]
```
**例题**：416. 分割等和子集

##### 10.2 字符串DP
**模板**：
```python
def longest_common_subsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```
**例题**：1143. 最长公共子序列

#### 11. 图遍历模式

##### 11.1 岛屿问题
**模板**：
```python
def num_islands(grid):
    def dfs(i, j):
        if 0<=i<len(grid) and 0<=j<len(grid[0]) and grid[i][j] == '1':
            grid[i][j] = '0'
            dfs(i+1, j); dfs(i-1, j); dfs(i, j+1); dfs(i, j-1)
    
    count = 0
    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] == '1':
                count += 1
                dfs(i, j)
    return count
```
**例题**：200. 岛屿数量

##### 11.2 拓扑排序
**模板**：
```python
def topological_sort(numCourses, prerequisites):
    graph = [[] for _ in range(numCourses)]
    indegree = [0] * numCourses
    for dest, src in prerequisites:
        graph[src].append(dest)
        indegree[dest] += 1
    
    queue = deque(i for i in range(numCourses) if indegree[i] == 0)
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)
    return order if len(order) == numCourses else []
```
**例题**：207. 课程表

#### 12. 位操作模式

##### 12.1 单一数字
**模板**：
```python
def single_number(nums):
    result = 0
    for num in nums:
        result ^= num
    return result
```
**例题**：136. 只出现一次的数字

##### 12.2 位计数
**模板**：
```python
def count_bits(n):
    result = [0]*(n+1)
    for i in range(1, n+1):
        result[i] = result[i & (i-1)] + 1
    return result
```
**例题**：338. 比特位计数

#### 13. 单调栈模式

##### 13.1 下一个更大元素
**模板**：
```python
def next_greater_element(nums):
    stack = []
    result = [-1] * len(nums)
    for i in range(len(nums)):
        while stack and nums[i] > nums[stack[-1]]:
            idx = stack.pop()
            result[idx] = nums[i]
        stack.append(i)
    return result
```
**例题**：496. 下一个更大元素 I

##### 13.2 柱状图最大矩形
**模板**：
```python
def largest_rectangle_area(heights):
    stack = []
    max_area = 0
    heights.append(0)
    for i in range(len(heights)):
        while stack and heights[i] < heights[stack[-1]]:
            h = heights[stack.pop()]
            w = i if not stack else i - stack[-1] - 1
            max_area = max(max_area, h * w)
        stack.append(i)
    return max_area
```
**例题**：84. 柱状图中最大的矩形

掌握这些模式后，LeetCode 解题可以转化为模式识别和模板应用的过程。建议通过分类练习强化每种模式，并注意不同模式间的组合应用。