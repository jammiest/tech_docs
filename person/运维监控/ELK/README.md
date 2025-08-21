# ELK 技术栈详解

ELK 是 Elasticsearch、Logstash 和 Kibana 三个开源工具的首字母缩写，构成了一个强大的日志管理和分析平台。

## 1. 核心组件架构

```
数据源 -> Logstash/Beats -> Elasticsearch -> Kibana
           (收集/处理)       (存储/搜索)     (可视化)
```

### 组件功能对比

| 组件 | 角色 | 主要功能 | 替代方案 |
|------|------|----------|----------|
| Elasticsearch | 搜索引擎 | 分布式搜索和分析 | OpenSearch, Solr |
| Logstash | 数据处理管道 | 收集、转换和发送数据 | Fluentd, Vector |
| Kibana | 可视化平台 | 数据探索和可视化 | Grafana, OpenSearch Dashboards |
| Beats | 轻量级数据收集器 | 单一用途数据采集 | Fluent Bit, Telegraf |

## 2. Elasticsearch 深度解析

### 核心概念

- **索引(Index)**: 类似数据库中的表
- **文档(Document)**: 类似表中的行，JSON格式
- **分片(Shard)**: 数据水平分割单元
- **副本(Replica)**: 分片的复制，提供高可用

### 分片分配公式

$$
\text{总分片数} = \text{主分片数} \times (1 + \text{副本数})
$$

### 常用API示例

```bash
# 创建索引
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}

# 查询文档
GET /my_index/_search
{
  "query": {
    "match": {
      "message": "error"
    }
  }
}

# 聚合分析
GET /logs/_search
{
  "aggs": {
    "status_count": {
      "terms": {
        "field": "status.keyword"
      }
    }
  }
}
```

## 3. Logstash 配置详解

### 典型配置文件结构

```ruby
input {
  # 输入插件，如file, beats, kafka等
  file {
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
  }
}

filter {
  # 数据处理插件
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  
  date {
    match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
  
  mutate {
    remove_field => [ "timestamp" ]
  }
}

output {
  # 输出插件
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "nginx-access-%{+YYYY.MM.dd}"
  }
}
```

### 性能优化技巧

1. **批量处理**：增大 `pipeline.batch.size` (默认125)
2. **工作线程**：调整 `pipeline.workers` 为CPU核心数
3. **JVM调优**：设置 `-Xms` 和 `-Xmx` 为相同值，避免动态调整

## 4. Kibana 高级功能

### 可视化类型矩阵

| 图表类型 | 适用场景 | 优势 |
|----------|----------|------|
| 柱状图 | 比较分类数据 | 直观对比 |
| 折线图 | 时间序列趋势 | 显示变化 |
| 饼图 | 比例分布 | 整体占比 |
| 热力图 | 密度分布 | 空间模式 |
| 标记地图 | 地理数据 | 位置分析 |

### Lens 可视化示例

```
1. 选择索引模式
2. 拖拽字段到可视化区域
3. 选择图表类型
4. 添加筛选条件
5. 保存可视化
```

## 5. Beats 家族

### Beats 组件比较

| Beat类型 | 数据源 | 典型用途 |
|----------|--------|----------|
| Filebeat | 日志文件 | 收集应用日志 |
| Metricbeat | 系统指标 | 服务器监控 |
| Packetbeat | 网络流量 | 网络分析 |
| Auditbeat | 审计数据 | 安全监控 |
| Heartbeat | 可用性 | 服务探测 |

### Filebeat 配置示例

```yaml
filebeat.inputs:
- type: log
  paths:
    - /var/log/app/*.log
  fields:
    app: myapp
    env: prod

output.elasticsearch:
  hosts: ["http://es-server:9200"]
  indices:
    - index: "app-logs-%{[fields.env]}-%{+yyyy.MM.dd}"
      when.equals:
        fields.app: "myapp"
```

## 6. 集群部署方案

### 生产环境架构

```
                      Load Balancer
                          |
       +------------------+------------------+
       |                  |                  |
[ Elasticsearch Node 1 ] [ Elasticsearch Node 2 ] [ Elasticsearch Node 3 ]
       | (Master eligible) | (Master eligible) | (Master eligible)
       |
[ Logstash Workers ] ---> [ Kafka ] <--- [ Beats ]
       |
[ Kibana ] [ APM Server ]
```

### 容量规划公式

$$
\text{所需存储} = \frac{\text{日志量/天} \times \text{保留天数} \times \text{压缩比}}{1 - \text{副本数比例}}
$$

其中压缩比通常为0.5-0.7

## 7. 性能优化指南

### Elasticsearch 调优参数

```yaml
# elasticsearch.yml
thread_pool.search.size: 2 * CPU核心数
thread_pool.search.queue_size: 1000

indices.fielddata.cache.size: 30%
indices.queries.cache.size: 10%

bootstrap.memory_lock: true
```

### JVM 配置建议

```
-Xms8g -Xmx8g  # 不超过物理内存50%
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

## 8. 安全配置

### 安全层实施

1. **网络层**：防火墙规则，VPC隔离
2. **传输层**：TLS加密通信
3. **认证层**：用户名/密码，LDAP集成
4. **授权层**：基于角色的访问控制(RBAC)
5. **审计层**：记录所有访问和操作

### 安全配置示例

```bash
# 启用基础安全
bin/elasticsearch-keystore create
bin/elasticsearch-setup-passwords auto

# 配置Kibana安全
xpack.security.enabled: true
xpack.security.audit.enabled: true
```

## 9. 监控与维护

### 关键监控指标

| 类别 | 指标 | 健康阈值 |
|------|------|----------|
| 集群健康 | status | green/yellow |
| 节点 | heap_used_percent | <75% |
| 索引 | search_latency | <100ms |
| 分片 | unassigned_shards | 0 |

### 常用维护命令

```bash
# 查看集群健康
GET /_cluster/health

# 清理缓存
POST /_cache/clear

# 强制合并段
POST /my_index/_forcemerge?max_num_segments=1

# 索引轮换策略
PUT /_ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

## 10. 常见问题解决方案

### 性能问题排查流程

```
1. 检查集群健康状态
2. 查看节点资源使用率(CPU,内存,磁盘I/O)
3. 分析慢查询日志
4. 检查索引分片分布
5. 评估查询和索引模式
6. 调整配置参数
```

### 错误处理指南

| 错误 | 可能原因 | 解决方案 |
|------|----------|----------|
| 429 Too Many Requests | 限流触发 | 增加队列大小或降低索引速率 |
| 503 Service Unavailable | 主节点选举问题 | 检查网络和节点状态 |
| 403 Forbidden | 权限不足 | 检查RBAC设置 |
| 400 Bad Request | 查询语法错误 | 验证查询DSL |

ELK 技术栈是现代可观测性体系的核心组件，通过合理配置和优化，可以构建高效、可靠的日志管理和分析平台，满足从开发调试到生产监控的全场景需求。