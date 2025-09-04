### 贪心算法与动态规划（Greedy vs Dynamic Programming）架构解析

贪心算法和动态规划是解决最优化问题的两大核心范式，在算法设计中具有互补性和根本性差异。以下从理论基础、设计模式、应用场景到实现架构进行系统对比分析。

#### 1. 核心概念与数学基础

##### 1.1 贪心算法（Greedy Algorithm）
基于局部最优选择期望达到全局最优的解。其有效性依赖于**贪心选择性质**和**最优子结构性质**。

**贪心选择性质**：全局最优解可以通过一系列局部最优选择得到  
**最优子结构**：问题的最优解包含其子问题的最优解

数学表达：对于问题 \(P\)，若存在选择 \(a^* \in A\) 使得：
\[
f(S) = a^* \oplus f(S - a^*)
\]
其中 \(f(S)\) 为解函数，\(\oplus\) 为组合操作。

##### 1.2 动态规划（Dynamic Programming）
通过将问题分解为重叠子问题，并存储子问题解避免重复计算。核心是**状态定义**和**状态转移方程**。

**最优子结构**：问题的最优解由相关子问题的最优解组合而成  
**重叠子问题**：递归算法会反复求解相同子问题

状态转移方程一般形式：
\[
dp[i] = \min/\max\{f(dp[j]), \text{ for } j < i\}
\]
或
\[
dp[i][j] = g(dp[i-1][j], dp[i][j-1], \ldots)
\]

#### 2. 算法设计模式对比

##### 2.1 贪心算法设计流程
1. 将问题转化为一系列选择步骤
2. 证明贪心选择性质（通常使用交换论证）
3. 证明最优子结构性质
4. 实现贪心选择策略

##### 2.2 动态规划设计流程
1. 定义状态 \(dp[i]\) 或 \(dp[i][j]\)
2. 确定状态转移方程
3. 初始化边界条件
4. 确定计算顺序（自底向上或记忆化搜索）
5. 构造最优解（需要时记录选择路径）

#### 3. 经典问题对比分析

##### 3.1 背包问题
**0-1背包问题**（动态规划）：
\[
dp[i][w] = \max(dp[i-1][w], dp[i-1][w-w_i] + v_i)
\]

**分数背包问题**（贪心算法）：
按价值密度 \(v_i/w_i\) 降序选择

##### 3.2 活动选择问题
贪心算法选择结束时间最早的活动：
```python
def activity_selection(s, f):
    n = len(f)
    selected = []
    i = 0
    selected.append(i)
    for j in range(1, n):
        if s[j] >= f[i]:
            selected.append(j)
            i = j
    return selected
```

##### 3.3 最长公共子序列（LCS）
动态规划状态转移：
\[
dp[i][j] = 
\begin{cases} 
dp[i-1][j-1] + 1 & \text{if } X[i] = Y[j] \\
\max(dp[i-1][j], dp[i][j-1]) & \text{otherwise}
\end{cases}
\]

#### 4. 性能与复杂度分析

| 特性 | 贪心算法 | 动态规划 |
|------|---------|----------|
| 时间复杂度 | 通常 \(O(n\log n)\) 或 \(O(n)\) | 通常 \(O(n^2)\) 或 \(O(n \cdot W)\) |
| 空间复杂度 | \(O(1)\) 或 \(O(n)\) | \(O(n)\) 或 \(O(n^2)\) |
| 解的质量 | 不一定最优（除非证明） | 保证最优解 |
| 适用问题 | 优化问题、调度问题 | 序列问题、路径问题 |

#### 5. 实现架构模式

##### 5.1 贪心算法通用模板
```python
def greedy_algorithm(inputs):
    # 预处理（如排序）
    sorted_inputs = sort(inputs, key=criterion)
    
    solution = []
    current_state = initial_state
    
    for item in sorted_inputs:
        if feasible(solution, item, current_state):
            solution.append(item)
            current_state = update_state(current_state, item)
    
    return solution
```

##### 5.2 动态规划通用模板
**自底向上迭代**：
```python
def dp_bottom_up(n):
    dp = [0] * (n+1)
    dp[0] = base_case
    
    for i in range(1, n+1):
        dp[i] = compute_from_previous(dp, i)
    
    return dp[n]
```

**记忆化搜索**：
```python
def dp_memoization(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    
    if n == 0:
        return base_case
    
    result = compute_from_subproblems(n, dp_memoization, memo)
    memo[n] = result
    return result
```

#### 6. 混合策略与进阶技术

##### 6.1 贪心启发式动态规划
对于状态空间过大的问题，使用贪心策略减少状态数：
- 状态剪枝：只保留"有希望"的状态
- 束搜索（Beam Search）：每步只保留前k个最优状态

##### 6.2 动态规划优化
- **斜率优化**：对于形如 \(dp[i] = \min\{f(j) + g(i-j)\}\) 的问题
- **四边形不等式**：优化区间DP决策单调性
- **状态压缩**：使用位运算减少空间复杂度

#### 7. 应用场景指南

**使用贪心算法当**：
- 问题具有贪心选择性质（如Huffman编码、最小生成树）
- 需要快速近似解且最优性可接受
- 问题规模极大，精确解不可行

**使用动态规划当**：
- 问题具有重叠子问题（如斐波那契数列、编辑距离）
- 需要精确最优解
- 问题可以自然分解为序列决策

#### 8. 正确性证明模式

**贪心算法证明**：
1. 证明贪心选择存在于某个最优解中
2. 证明剩余子问题具有最优子结构

**动态规划证明**：
1. 证明最优子结构性质
2. 证明状态转移方程的正确性
3. 证明边界条件的正确性

贪心算法和动态规划代表了算法设计中"局部决策"与"全局规划"的两种根本哲学。在实际系统设计中，常需要结合两者优势：用贪心策略快速缩小搜索空间，再用动态规划求解剩余问题，形成分层优化架构。