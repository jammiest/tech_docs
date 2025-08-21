# 图算法知识梳理

图算法是计算机科学中的重要组成部分，广泛应用于网络分析、路径规划、社交网络等领域。

## 1. 图的基本表示方法

### 1.1 邻接矩阵 (Adjacency Matrix)
- **空间复杂度**：$O(V^2)$
- **适用场景**：稠密图，需要快速判断两顶点是否相邻

```python
# 无向图的邻接矩阵表示
adj_matrix = [
    [0, 1, 1, 0],
    [1, 0, 1, 1],
    [1, 1, 0, 0],
    [0, 1, 0, 0]
]
```

### 1.2 邻接表 (Adjacency List)
- **空间复杂度**：$O(V + E)$
- **适用场景**：稀疏图，节省空间

```python
# 有向图的邻接表表示
adj_list = {
    0: [1, 2],
    1: [2, 3],
    2: [0],
    3: [1]
}
```

## 2. 图的遍历算法

### 2.1 深度优先搜索 (DFS)
- **时间复杂度**：$O(V + E)$
- **空间复杂度**：$O(V)$
- **应用**：拓扑排序、连通分量、路径查找

```python
def dfs(graph, start):
    visited = set()
    stack = [start]
    
    while stack:
        vertex = stack.pop()
        if vertex not in visited:
            visited.add(vertex)
            stack.extend(reversed(graph[vertex] - visited))
    return visited
```

### 2.2 广度优先搜索 (BFS)
- **时间复杂度**：$O(V + E)$
- **空间复杂度**：$O(V)$
- **应用**：最短路径(无权图)、社交网络中的"好友推荐"

```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue = deque([start])
    
    while queue:
        vertex = queue.popleft()
        if vertex not in visited:
            visited.add(vertex)
            queue.extend(graph[vertex] - visited)
    return visited
```

## 3. 最短路径算法

### 3.1 Dijkstra算法
- **原理**：贪心算法，适用于有向/无向带权图(非负权值)
- **时间复杂度**：
  - 普通实现：$O(V^2)$
  - 优先队列实现：$O(E + V \log V)$
- **空间复杂度**：$O(V)$

```python
import heapq

def dijkstra(graph, start):
    distances = {vertex: float('infinity') for vertex in graph}
    distances[start] = 0
    pq = [(0, start)]
    
    while pq:
        current_distance, current_vertex = heapq.heappop(pq)
        
        if current_distance > distances[current_vertex]:
            continue
            
        for neighbor, weight in graph[current_vertex].items():
            distance = current_distance + weight
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))
    
    return distances
```

### 3.2 Bellman-Ford算法
- **原理**：动态规划，适用于含负权边的图
- **时间复杂度**：$O(VE)$
- **空间复杂度**：$O(V)$
- **特点**：能检测负权环

```python
def bellman_ford(graph, start):
    distances = {vertex: float('infinity') for vertex in graph}
    distances[start] = 0
    
    for _ in range(len(graph) - 1):
        for vertex in graph:
            for neighbor, weight in graph[vertex].items():
                if distances[vertex] + weight < distances[neighbor]:
                    distances[neighbor] = distances[vertex] + weight
    
    # 检查负权环
    for vertex in graph:
        for neighbor, weight in graph[vertex].items():
            if distances[vertex] + weight < distances[neighbor]:
                return "Graph contains negative weight cycle"
    
    return distances
```

### 3.3 Floyd-Warshall算法
- **原理**：动态规划，计算所有顶点对的最短路径
- **时间复杂度**：$O(V^3)$
- **空间复杂度**：$O(V^2)$
- **特点**：能处理负权边(但不能有负权环)

```python
def floyd_warshall(graph):
    dist = {v: {u: float('infinity') for u in graph} for v in graph}
    
    for v in graph:
        dist[v][v] = 0
        for u, w in graph[v].items():
            dist[v][u] = w
    
    for k in graph:
        for i in graph:
            for j in graph:
                if dist[i][j] > dist[i][k] + dist[k][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    
    return dist
```

### 3.4 A*搜索算法
- **原理**：启发式搜索，结合Dijkstra和贪心算法
- **时间复杂度**：取决于启发函数，最坏$O(b^d)$，b为分支因子，d为深度
- **空间复杂度**：$O(V)$
- **应用**：路径规划、游戏AI

```python
def heuristic(a, b):
    return abs(a[0] - b[0]) + abs(a[1] - b[1])

def a_star(graph, start, goal):
    open_set = {start}
    came_from = {}
    g_score = {vertex: float('infinity') for vertex in graph}
    g_score[start] = 0
    f_score = {vertex: float('infinity') for vertex in graph}
    f_score[start] = heuristic(start, goal)
    
    while open_set:
        current = min(open_set, key=lambda vertex: f_score[vertex])
        if current == goal:
            path = []
            while current in came_from:
                path.append(current)
                current = came_from[current]
            path.append(start)
            path.reverse()
            return path
            
        open_set.remove(current)
        
        for neighbor in graph[current]:
            tentative_g_score = g_score[current] + graph[current][neighbor]
            if tentative_g_score < g_score[neighbor]:
                came_from[neighbor] = current
                g_score[neighbor] = tentative_g_score
                f_score[neighbor] = g_score[neighbor] + heuristic(neighbor, goal)
                if neighbor not in open_set:
                    open_set.add(neighbor)
    
    return None
```

## 4. 最小生成树算法

### 4.1 Kruskal算法
- **原理**：贪心算法，按边权排序逐步添加不形成环的边
- **时间复杂度**：$O(E \log E)$ (主要来自排序)
- **空间复杂度**：$O(E + V)$
- **特点**：适合稀疏图

```python
class UnionFind:
    def __init__(self, size):
        self.parent = list(range(size))
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        x_root = self.find(x)
        y_root = self.find(y)
        if x_root != y_root:
            self.parent[y_root] = x_root

def kruskal(graph):
    edges = []
    for u in graph:
        for v, w in graph[u].items():
            edges.append((w, u, v))
    edges.sort()
    
    uf = UnionFind(len(graph))
    mst = []
    
    for w, u, v in edges:
        if uf.find(u) != uf.find(v):
            uf.union(u, v)
            mst.append((u, v, w))
    
    return mst
```

### 4.2 Prim算法
- **原理**：贪心算法，从任意顶点开始逐步扩展最小生成树
- **时间复杂度**：
  - 邻接矩阵：$O(V^2)$
  - 优先队列+邻接表：$O(E \log V)$
- **空间复杂度**：$O(V + E)$
- **特点**：适合稠密图

```python
import heapq

def prim(graph, start):
    mst = []
    visited = set([start])
    edges = [
        (weight, start, to)
        for to, weight in graph[start].items()
    ]
    heapq.heapify(edges)
    
    while edges:
        weight, u, v = heapq.heappop(edges)
        if v not in visited:
            visited.add(v)
            mst.append((u, v, weight))
            for to, w in graph[v].items():
                if to not in visited:
                    heapq.heappush(edges, (w, v, to))
    
    return mst
```

## 5. 网络流算法

### 5.1 Ford-Fulkerson方法
- **原理**：通过增广路径逐步增加流量
- **时间复杂度**：$O(E \cdot max_flow)$
- **空间复杂度**：$O(V + E)$
- **应用**：最大流问题

```python
def bfs_for_ff(residual_graph, s, t, parent):
    visited = {v: False for v in residual_graph}
    queue = [s]
    visited[s] = True
    
    while queue:
        u = queue.pop(0)
        for v in residual_graph[u]:
            if not visited[v] and residual_graph[u][v] > 0:
                queue.append(v)
                visited[v] = True
                parent[v] = u
                if v == t:
                    return True
    return False

def ford_fulkerson(graph, source, sink):
    residual_graph = {u: {v: graph[u][v] for v in graph[u]} for u in graph}
    parent = {}
    max_flow = 0
    
    while bfs_for_ff(residual_graph, source, sink, parent):
        path_flow = float('infinity')
        s = sink
        while s != source:
            path_flow = min(path_flow, residual_graph[parent[s]][s])
            s = parent[s]
        
        max_flow += path_flow
        v = sink
        while v != source:
            u = parent[v]
            residual_graph[u][v] -= path_flow
            residual_graph[v][u] += path_flow
            v = u
    
    return max_flow
```

### 5.2 Edmonds-Karp算法
- **原理**：Ford-Fulkerson方法的BFS实现，保证多项式时间复杂度
- **时间复杂度**：$O(V E^2)$
- **空间复杂度**：$O(V + E)$

## 6. 图匹配算法

### 6.1 匈牙利算法
- **原理**：寻找二分图的最大匹配
- **时间复杂度**：$O(V E)$
- **空间复杂度**：$O(V + E)$

```python
def bpm(graph, u, match_to, visited):
    for v in graph[u]:
        if not visited[v]:
            visited[v] = True
            if match_to[v] == -1 or bpm(graph, match_to[v], match_to, visited):
                match_to[v] = u
                return True
    return False

def hungarian(graph, num_jobs):
    match_to = [-1] * num_jobs
    result = 0
    
    for u in graph:
        visited = [False] * num_jobs
        if bpm(graph, u, match_to, visited):
            result += 1
    
    return result
```

## 7. 图算法比较

| 算法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|------------|------------|----------|
| DFS | $O(V + E)$ | $O(V)$ | 拓扑排序、连通分量 |
| BFS | $O(V + E)$ | $O(V)$ | 无权图最短路径 |
| Dijkstra | $O(E + V \log V)$ | $O(V)$ | 非负权图最短路径 |
| Bellman-Ford | $O(VE)$ | $O(V)$ | 含负权边的最短路径 |
| Floyd-Warshall | $O(V^3)$ | $O(V^2)$ | 所有顶点对最短路径 |
| A* | $O(b^d)$ | $O(V)$ | 启发式路径搜索 |
| Kruskal | $O(E \log E)$ | $O(E + V)$ | 稀疏图最小生成树 |
| Prim | $O(E \log V)$ | $O(V + E)$ | 稠密图最小生成树 |
| Ford-Fulkerson | $O(E \cdot max_flow)$ | $O(V + E)$ | 网络最大流 |
| Edmonds-Karp | $O(V E^2)$ | $O(V + E)$ | 网络最大流(多项式时间) |
| 匈牙利算法 | $O(V E)$ | $O(V + E)$ | 二分图匹配 |

## 8. 图算法的选择建议

1. **遍历图**：
   - 需要深度优先的特性：DFS
   - 需要广度优先的特性：BFS

2. **最短路径**：
   - 无权图：BFS
   - 带权图(无负权边)：Dijkstra
   - 带权图(有负权边)：Bellman-Ford
   - 所有顶点对最短路径：Floyd-Warshall

3. **最小生成树**：
   - 稀疏图：Kruskal
   - 稠密图：Prim

4. **网络流**：
   - 小规模网络：Ford-Fulkerson
   - 大规模网络：Edmonds-Karp或更高级的预流推进算法

5. **图匹配**：
   - 二分图匹配：匈牙利算法
   - 一般图匹配：开花算法(Blossom Algorithm)

## 9. 高级图算法

### 9.1 强连通分量 (Kosaraju算法)
- **时间复杂度**：$O(V + E)$
- **空间复杂度**：$O(V + E)$

```python
def kosaraju(graph):
    visited = set()
    order = []
    
    def dfs(u):
        stack = [(u, False)]
        while stack:
            node, processed = stack.pop()
            if processed:
                order.append(node)
                continue
            if node in visited:
                continue
            visited.add(node)
            stack.append((node, True))
            for v in graph.get(node, []):
                if v not in visited:
                    stack.append((v, False))
    
    for u in graph:
        if u not in visited:
            dfs(u)
    
    reversed_graph = {}
    for u in graph:
        for v in graph[u]:
            reversed_graph.setdefault(v, set()).add(u)
    
    visited = set()
    components = []
    
    for u in reversed(order):
        if u not in visited:
            component = []
            stack = [u]
            visited.add(u)
            while stack:
                node = stack.pop()
                component.append(node)
                for v in reversed_graph.get(node, []):
                    if v not in visited:
                        visited.add(v)
                        stack.append(v)
            components.append(component)
    
    return components
```

### 9.2 欧拉路径/回路 (Hierholzer算法)
- **时间复杂度**：$O(E)$
- **空间复杂度**：$O(E)$

```python
def hierholzer(graph):
    if len(graph) == 0:
        return []
    
    # 检查是否存在欧拉回路或路径
    in_degree = {}
    out_degree = {}
    for u in graph:
        out_degree[u] = len(graph[u])
        for v in graph[u]:
            in_degree[v] = in_degree.get(v, 0) + 1
    
    start_node = next(iter(graph))
    odd_nodes = 0
    for node in graph:
        if in_degree.get(node, 0) != out_degree.get(node, 0):
            odd_nodes += 1
            if out_degree.get(node, 0) > in_degree.get(node, 0):
                start_node = node
    
    if odd_nodes not in (0, 2):
        return None  # 不存在欧拉路径或回路
    
    stack = [start_node]
    path = []
    
    while stack:
        current = stack[-1]
        if graph[current]:
            next_node = graph[current].pop()
            stack.append(next_node)
        else:
            path.append(stack.pop())
    
    return path[::-1]
```

### 9.3 哈密尔顿回路问题
- **特点**：NP完全问题，没有已知的多项式时间算法
- **常见解法**：回溯法、动态规划(Held-Karp算法 $O(n^2 2^n)$)

## 10. 图算法在实际中的应用

1. **社交网络分析**：
   - 社区检测(连通分量、聚类算法)
   - 影响力分析(PageRank算法)
   - 最短路径(六度分隔理论)

2. **交通网络**：
   - 路径规划(Dijkstra、A*)
   - 流量优化(网络流算法)

3. **计算机视觉**：
   - 图像分割(图割算法)
   - 物体识别(图匹配)

4. **生物信息学**：
   - 蛋白质相互作用网络
   - 基因调控网络分析

5. **推荐系统**：
   - 协同过滤(基于图的推荐)
   - 知识图谱




