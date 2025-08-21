# Prometheus 监控系统详解

Prometheus 是一个开源的系统监控和警报工具包，最初由 SoundCloud 开发，现在是 CNCF 毕业项目，已成为云原生监控的事实标准。

## 1. 核心架构

### 系统组件图

```
Prometheus Server
├─ Retrieval (拉取指标)
├─ Storage (时序数据库)
├─ PromQL (查询语言)
├─ Alertmanager (告警管理)
│
├─ Exporters (指标暴露)
│   ├─ Node Exporter
│   ├─ Blackbox Exporter
│   └─ Application Exporters
│
└─ Service Discovery
    ├─ Static Configs
    ├─ Kubernetes SD
    └─ Consul SD
```

### 工作流程

1. **指标采集**：通过HTTP定期从目标拉取指标
2. **存储**：本地时序数据库存储采集的数据
3. **处理**：通过PromQL查询和处理数据
4. **可视化**：通过Grafana等工具展示
5. **告警**：基于规则触发Alertmanager

## 2. 数据模型

### 指标组成要素

```
<metric name>{<label name>=<label value>, ...} <value> [timestamp]
```

示例：
```
http_requests_total{method="POST", handler="/api/users"} 1027 1599034314000
```

### 指标类型

| 类型 | 描述 | 用例 |
|------|------|------|
| Counter | 只增不减的计数器 | 请求数、错误数 |
| Gauge | 可增可减的仪表盘 | 内存使用、温度 |
| Histogram | 采样观察的分桶统计 | 响应时间分布 |
| Summary | 客户端计算的分位数 | 服务延迟 |

## 3. PromQL 查询语言

### 常用查询模式

```promql
# 计算每秒增长率
rate(http_requests_total[5m])

# 聚合计算
sum by (instance) (http_requests_total)

# 条件筛选
node_memory_MemFree_bytes / node_memory_MemTotal_bytes < 0.2

# 时间偏移比较
http_errors_total - http_errors_total offset 1h
```

### 运算符优先级

$$
\begin{aligned}
& \text{最高} \quad \text{^} \\
& \text{*, /, %} \\
& \text{+, -} \\
& \text{==, !=, <=, <, >=, >} \\
& \text{and, unless} \\
& \text{最低} \quad \text{or}
\end{aligned}
$$

## 4. 配置详解

### prometheus.yml 结构

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node1:9100', 'node2:9100']
    
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### 重标签配置流程

```
原始元数据 -> relabel_configs -> 最终标签
```

常用操作：
- `keep`/`drop`：过滤目标
- `replace`：修改标签值
- `labelmap`：重命名标签

## 5. 存储机制

### 本地存储结构

```
data/
├─ 01BKGV7BM0S3HZ91HHV6R5F6X/  # 块目录
│  ├─ meta.json                # 元数据
│  ├─ chunks/                  # 实际数据
│  │   └─ 000001               # 分块文件
│  └─ index                    # 索引文件
├─ wal/                        # 预写日志
│   ├─ 000000002               # WAL段
│   └─ 000000003
```

### 保留策略计算

$$
\text{保留时间} = \frac{\text{存储空间}}{\text{采样频率} \times \text{指标数量} \times \text{样本大小}}
$$

典型样本大小约1-2字节

## 6. 告警管理

### Alertmanager 路由树

```
全局配置
└─ 根路由
   ├─ 子路由1 (team=frontend)
   │   ├─ 子路由1.1 (severity=critical)
   │   └─ 子路由1.2 (severity=warning)
   └─ 子路由2 (team=backend)
       └─ 子路由2.1 (env=prod)
```

### 告警规则示例

```yaml
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 1
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: "High latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has high request latency ({{ $value }}s)"
```

## 7. 联邦与高可用

### 联邦架构

```
全局Prometheus
├─ 区域Prometheus A
│   ├─ 服务组1
│   └─ 服务组2
└─ 区域Prometheus B
    ├─ 服务组3
    └─ 服务组4
```

### HA 部署方案

```
                       Load Balancer
                           |
       +-------------------+-------------------+
       |                                       |
[Prometheus A]                          [Prometheus B]
       |                                       |
[Alertmanager Cluster]                [Same Data Sources]
```

## 8. 性能优化

### 内存使用公式

$$
\text{内存需求} \approx \frac{\text{活跃序列数} \times 2KB}{\text{压缩率}}
$$

### 关键调优参数

```yaml
# 存储配置
storage.tsdb.retention.time: 15d
storage.tsdb.wal-compression: true

# 查询限制
query.max-concurrency: 20
query.timeout: 2m

# 资源分配
--storage.tsdb.max-block-chunk-segment-size=512MB
```

## 9. 集成生态

### 常用Exporter矩阵

| Exporter | 监控目标 | 关键指标 |
|----------|----------|----------|
| node_exporter | 主机 | CPU,内存,磁盘,网络 |
| blackbox_exporter | 网络探测 | 可用性,延迟 |
| mysqld_exporter | MySQL | 查询性能,连接数 |
| kube-state-metrics | Kubernetes | 资源状态,Pod计数 |
| snmp_exporter | 网络设备 | 接口状态,流量 |

### 与Grafana集成

典型仪表板JSON配置包含：
- 面板布局
- 查询定义
- 可视化选项
- 告警阈值

## 10. 实战案例

### Kubernetes监控方案

```yaml
# kube-prometheus-stack 核心组件
- prometheus-operator
- prometheus
- alertmanager
- kube-state-metrics
- node-exporter
- grafana
```

### 黑盒监控示例

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: [200,301,302]
      no_follow_redirects: false
      preferred_ip_protocol: "ip4"
```

Prometheus 的强大之处在于其多维数据模型、灵活的查询语言和活跃的生态系统，使其成为云原生时代监控系统的首选解决方案。通过合理设计和配置，可以构建从基础设施到应用层的全方位监控体系。