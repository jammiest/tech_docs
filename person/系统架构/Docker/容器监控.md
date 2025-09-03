# Docker 容器监控完全指南

## 监控体系架构

```
+----------------+     +----------------+     +----------------+
|   数据采集层    | --> |   数据存储层    | --> |   可视化层     |
| - cAdvisor     |     | - Prometheus   |     | - Grafana      |
| - Node Exporter|     | - InfluxDB    |     | - Kibana       |
| - Docker Stats |     | - Elasticsearch|     | - Dashboard    |
+----------------+     +----------------+     +----------------+
```

## 基础监控命令

### Docker 内置监控
```bash
# 实时监控所有容器资源
docker stats

# 监控特定容器
docker stats nginx mysql redis

# 格式化输出监控数据
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# 仅显示容器ID和CPU使用率
docker stats --format "{{.ID}}: {{.CPUPerc}}"

# 无流模式（单次输出）
docker stats --no-stream

# 监控容器进程
docker top container_name

# 查看容器详细信息
docker inspect container_name
```

### 资源限制检查
```bash
# 检查容器资源限制
docker inspect -f '{{.HostConfig.Memory}}' container_name
docker inspect -f '{{.HostConfig.NanoCpus}}' container_name

# 查看实际资源使用
docker stats --format "table {{.Name}}\t{{.MemPerc}}\t{{.MemUsage}}"

# 检查容器退出状态
docker inspect -f '{{.State.ExitCode}}' container_name
```

## cAdvisor 监控部署

### 部署 cAdvisor
```bash
# 运行 cAdvisor 容器
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:latest

# 验证 cAdvisor
curl http://localhost:8080/metrics
```

### cAdvisor 监控指标
- **CPU**: 使用率、限制、时间片
- **内存**: 使用量、限制、缓存、交换
- **网络**: 输入输出流量、错误包
- **磁盘**: IOPS、吞吐量、使用量
- **容器**: 数量、状态、重启次数

## Prometheus 监控体系

### 部署 Prometheus
```yaml
# docker-compose-prometheus.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/console_templates'
    networks:
      - monitoring

volumes:
  prometheus_data:

networks:
  monitoring:
    driver: bridge
```

### Prometheus 配置
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['cadvisor:8080']
    metrics_path: /metrics
    scrape_interval: 5s

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

### Node Exporter 部署
```bash
# 运行 Node Exporter
docker run -d \
  --name=node-exporter \
  --net=host \
  --pid=host \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
```

## Grafana 可视化

### 部署 Grafana
```yaml
# docker-compose-grafana.yml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./dashboards:/var/lib/grafana/dashboards
    networks:
      - monitoring

volumes:
  grafana_data:

networks:
  monitoring:
    external: true
```

### 常用监控仪表板
- **Docker容器监控**: 193（官方模板）
- **Node Exporter**: 1860（系统监控）
- **cAdvisor**: 14282（容器详细监控）
- **自定义仪表板**: 根据业务需求定制

## 高级监控指标

### 关键性能指标（KPI）
```bash
# CPU 使用率监控
docker stats --format "{{.CPUPerc}}" container_name

# 内存使用监控
docker stats --format "{{.MemUsage}}" container_name

# 网络流量监控
docker exec container_name cat /proc/net/dev

# 磁盘 I/O 监控
docker exec container_name iostat -x 1
```

### 自定义监控脚本
```bash
#!/bin/bash
# 容器健康检查脚本
CONTAINERS=$(docker ps -q)

for container in $CONTAINERS; do
    name=$(docker inspect -f '{{.Name}}' $container | sed 's/\///')
    status=$(docker inspect -f '{{.State.Status}}' $container)
    health=$(docker inspect -f '{{.State.Health.Status}}' $container 2>/dev/null || echo "N/A")
    
    echo "Container: $name, Status: $status, Health: $health"
    
    # 检查资源使用
    cpu_usage=$(docker stats --no-stream --format "{{.CPUPerc}}" $container)
    mem_usage=$(docker stats --no-stream --format "{{.MemPerc}}" $container)
    
    echo "CPU: $cpu_usage, Memory: $mem_usage"
done
```

## 日志监控

### ELK Stack 集成
```yaml
# docker-compose-elk.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    environment:
      - discovery.type=single-node
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  logstash:
    image: docker.elastic.co/logstash/logstash:7.14.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.14.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch_data:
```

### 日志收集配置
```bash
# 使用 Filebeat 收集容器日志
docker run -d \
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" \
  --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
  docker.elastic.co/beats/filebeat:7.14.0
```

## 告警系统

### Prometheus Alertmanager
```yaml
# docker-compose-alertmanager.yml
version: '3.8'

services:
  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    networks:
      - monitoring

networks:
  monitoring:
    external: true
```

### 告警规则配置
```yaml
# alert.rules.yml
groups:
- name: docker-alerts
  rules:
  - alert: HighCPUUsage
    expr: rate(container_cpu_usage_seconds_total[5m]) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.container }}"
      description: "CPU usage is above 80% for 5 minutes"

  - alert: HighMemoryUsage
    expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High memory usage on {{ $labels.container }}"
      description: "Memory usage is above 90% for 5 minutes"
```

## 健康检查监控

### 容器健康检查配置
```yaml
# docker-compose-healthcheck.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

  api:
    image: myapp:latest
    healthcheck:
      test: ["CMD-SHELL", "wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 5
```

### 健康状态监控
```bash
# 检查所有容器健康状态
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Health}}"

# 监控健康状态变化
watch -n 5 'docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Health}}"'

# 获取健康检查日志
docker inspect --format='{{json .State.Health}}' container_name
```

## 性能分析工具

### 性能调试工具
```bash
# 使用 perf 分析性能
docker run -it --rm --privileged --pid=host alpine \
  sh -c 'apk add perf && perf top'

# 使用 strace 跟踪系统调用
docker run -it --rm --cap-add=SYS_PTRACE alpine \
  sh -c 'apk add strace && strace -p 1'

# 使用 tcpdump 抓包
docker run -it --rm --net=host -v $(pwd):/data alpine \
  sh -c 'apk add tcpdump && tcpdump -i any -w /data/capture.pcap'
```

### 资源使用分析
```bash
# 分析容器内存使用
docker exec container_name cat /proc/meminfo

# 分析容器 CPU 使用
docker exec container_name mpstat 1

# 分析磁盘 I/O
docker exec container_name iostat -x 1

# 分析网络连接
docker exec container_name netstat -tuln
```

## 自动化监控脚本

### 监控告警脚本
```bash
#!/bin/bash
# 容器监控告警脚本
THRESHOLD_CPU=80
THRESHOLD_MEM=90
THRESHOLD_DISK=85

# 监控 CPU
docker stats --no-stream --format "{{.Name}} {{.CPUPerc}}" | \
while read name cpu; do
  cpu_num=$(echo $cpu | sed 's/%//')
  if [ $(echo "$cpu_num > $THRESHOLD_CPU" | bc) -eq 1 ]; then
    echo "ALERT: High CPU usage in $name: $cpu"
    # 发送告警通知
  fi
done

# 监控内存
docker stats --no-stream --format "{{.Name}} {{.MemPerc}}" | \
while read name mem; do
  mem_num=$(echo $mem | sed 's/%//')
  if [ $(echo "$mem_num > $THRESHOLD_MEM" | bc) -eq 1 ]; then
    echo "ALERT: High Memory usage in $name: $mem"
    # 发送告警通知
  fi
done
```

### 定期健康检查
```bash
#!/bin/bash
# 定期健康检查脚本
LOG_FILE="/var/log/container-health.log"

{
  echo "=== Container Health Check $(date) ==="
  
  # 检查容器状态
  docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.State}}\t{{.Ports}}"
  
  echo ""
  echo "=== Resource Usage ==="
  docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemPerc}}\t{{.NetIO}}\t{{.BlockIO}}"
  
  echo ""
  echo "=== Health Status ==="
  docker ps --format "{{.Names}}" | while read container; do
    health=$(docker inspect --format='{{.State.Health.Status}}' $container 2>/dev/null || echo "N/A")
    echo "$container: $health"
  done
  
} >> $LOG_FILE
```

> 提示：建立完整的监控体系需要结合多种工具，根据实际业务需求选择合适的监控方案。

!> 重要：生产环境监控应设置适当的告警阈值，并确保告警通知能够及时送达相关人员。