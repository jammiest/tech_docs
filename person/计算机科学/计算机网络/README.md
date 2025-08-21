# 计算机网络概述

计算机网络是指将地理位置不同的具有独立功能的多台计算机及其外部设备，通过通信线路连接起来，在网络操作系统、网络管理软件及网络通信协议的管理和协调下，实现资源共享和信息传递的系统。

## 网络体系结构

### 1. OSI七层模型
| 层次 | 名称         | 功能描述                     | 典型协议/设备       |
|------|--------------|----------------------------|-------------------|
| 7    | 应用层        | 用户接口，提供网络服务         | HTTP, FTP, SMTP   |
| 6    | 表示层        | 数据格式转换、加密解密         | SSL, TLS          |
| 5    | 会话层        | 建立、管理和终止会话           | NetBIOS, RPC      |
| 4    | 传输层        | 端到端连接，可靠传输           | TCP, UDP          |
| 3    | 网络层        | 路径选择、IP寻址              | IP, ICMP, 路由器    |
| 2    | 数据链路层     | 帧传输、MAC寻址、差错控制      | Ethernet, PPP, 交换机 |
| 1    | 物理层        | 比特传输、物理连接             | RS-232, 网卡, 集线器 |

### 2. TCP/IP四层模型
- **应用层**：对应OSI的应用层、表示层、会话层
- **传输层**：TCP、UDP
- **网络层**：IP、ICMP
- **网络接口层**：对应OSI的数据链路层和物理层

## 关键协议与技术

### 1. IP协议
- IPv4地址：32位，点分十进制表示（如192.168.1.1）
- IPv6地址：128位，冒号分隔十六进制表示（如2001:0db8:85a3::8a2e:0370:7334）
- 子网划分：$$可用主机数 = 2^{(32-网络前缀长度)} - 2$$

### 2. TCP协议
- 三次握手建立连接：
  1. SYN
  2. SYN-ACK
  3. ACK
- 四次挥手终止连接：
  1. FIN
  2. ACK
  3. FIN
  4. ACK
- 流量控制：滑动窗口机制
- 拥塞控制算法：
  - 慢启动
  - 拥塞避免
  - 快速重传
  - 快速恢复

### 3. HTTP协议
- 请求方法：GET, POST, PUT, DELETE等
- 状态码：
  - 1xx：信息响应
  - 2xx：成功（200 OK）
  - 3xx：重定向（301 Moved Permanently）
  - 4xx：客户端错误（404 Not Found）
  - 5xx：服务器错误（500 Internal Server Error）
- HTTP/2：多路复用、头部压缩、服务器推送
- HTTPS：HTTP over TLS/SSL

## 网络性能指标

1. **带宽(Bandwidth)**：理论最大数据传输速率（bps）
2. **吞吐量(Throughput)**：实际数据传输速率
3. **延迟(Latency)**：
   - 传播延迟：$$T_{prop} = \frac{距离}{传播速度}$$
   - 传输延迟：$$T_{trans} = \frac{报文长度}{带宽}$$
   - 总延迟：$$T_{total} = T_{prop} + T_{trans} + T_{queue} + T_{proc}$$
4. **抖动(Jitter)**：延迟的变化量
5. **丢包率(Packet Loss Rate)**：$$丢包率 = \frac{丢失分组数}{总发送分组数} \times 100\%$$

## 路由算法

### 1. 距离向量算法
- 每个路由器维护到所有目的地的距离向量
- 定期与邻居交换路由信息
- Bellman-Ford方程：$$D_x(y) = min_v\{c(x,v) + D_v(y)\}$$

### 2. 链路状态算法
- 每个路由器构建整个网络的拓扑图
- 使用Dijkstra算法计算最短路径
- Dijkstra算法伪代码：
  ```python
  def dijkstra(graph, start):
      distances = {node: float('infinity') for node in graph}
      distances[start] = 0
      pq = PriorityQueue()
      pq.put((0, start))
      
      while not pq.empty():
          current_distance, current_node = pq.get()
          if current_distance > distances[current_node]:
              continue
          for neighbor, weight in graph[current_node].items():
              distance = current_distance + weight
              if distance < distances[neighbor]:
                  distances[neighbor] = distance
                  pq.put((distance, neighbor))
      return distances
  ```

## 网络安全

### 1. 加密技术
- 对称加密：AES, DES
  $$C = E(K, P), P = D(K, C)$$
- 非对称加密：RSA, ECC
  $$C = E(PK, P), P = D(SK, C)$$
- 哈希函数：SHA-256, MD5
  $$h = H(M)$$

### 2. 安全协议
- SSL/TLS：用于HTTPS
- IPsec：网络层安全
- WPA2/WPA3：无线网络安全

## 网络设备比较

| 设备       | OSI层 | 功能特点                     | 转发依据        |
|------------|-------|----------------------------|---------------|
| 集线器      | 物理层 | 广播所有端口                  | 无，简单转发     |
| 交换机      | 数据链路层 | 学习MAC地址，单播转发          | MAC地址表      |
| 路由器      | 网络层 | 连接不同网络，路径选择          | 路由表         |
| 网关        | 应用层 | 协议转换，连接不同体系网络       | 应用协议       |

## 无线网络技术

1. **Wi-Fi标准**：
   - 802.11a/b/g/n/ac/ax
   - 频段：2.4GHz, 5GHz, 6GHz
2. **蜂窝网络**：
   - 2G/3G/4G LTE/5G
3. **蓝牙**：短距离个人区域网络
4. **Zigbee**：低功耗物联网协议

计算机网络是现代信息技术的基础，理解网络原理和协议对于开发分布式系统、云计算应用和网络安全解决方案至关重要。随着5G、物联网和边缘计算的发展，网络技术仍在持续演进。