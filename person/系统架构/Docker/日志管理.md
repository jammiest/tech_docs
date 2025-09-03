# Docker 日志管理完全指南

## 日志驱动体系

### 支持的日志驱动
| 日志驱动 | 描述 | 适用场景 | 性能 |
|---------|------|---------|------|
| **json-file** | 默认驱动，JSON格式文件 | 开发测试 | 中等 |
| **journald** | systemd 日志系统 | systemd 环境 | 高 |
| **syslog** | Syslog 协议 | 企业日志系统 | 高 |
| **gelf** | Graylog Extended Format | Graylog 集成 | 高 |
| **fluentd** | Fluentd 收集器 | 大规模部署 | 高 |
| **awslogs** | AWS CloudWatch | AWS 环境 | 高 |
| **splunk** | Splunk 集成 | Splunk 用户 | 高 |
| **etw** | Windows事件跟踪 | Windows容器 | - |
| **none** | 禁用日志 | 特殊需求 | - |

## 基础日志操作

### 日志查看命令
```bash
# 查看容器日志
docker logs container_name

# 实时跟踪日志
docker logs -f container_name

# 查看最后N行日志
docker logs --tail=100 container_name

# 查看特定时间后的日志
docker logs --since="2023-01-01T00:00:00" container_name

# 查看时间范围内的日志
docker logs --since="2023-01-01" --until="2023-01-02" container_name

# 显示时间戳
docker logs -t container_name

# 显示日志详情
docker logs --details container_name
```

### 日志驱动配置
```bash
# 运行容器时指定日志驱动
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx:alpine

# 使用 syslog 驱动
docker run -d \
  --log-driver syslog \
  --log-opt syslog-address=udp://syslog-server:514 \
  --log-opt tag="{{.Name}}" \
  nginx:alpine
```

## 全局日志配置

### Daemon 配置
```json
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "production",
    "env": "os,customer"
  }
}
```

### 配置验证与重载
```bash
# 检查当前日志配置
docker info --format '{{.LoggingDriver}}'

# 重载 Docker 配置
sudo systemctl reload docker

# 验证配置生效
docker run --rm alpine echo "test log" && docker logs <container_id>
```

## 日志驱动详解

### json-file 驱动（默认）
```bash
# 自定义 json-file 配置
docker run -d \
  --log-driver json-file \
  --log-opt max-size=50m \
  --log-opt max-file=5 \
  --log-opt compress=true \
  --log-opt labels=env,app \
  nginx:alpine

# 日志文件位置
/var/lib/docker/containers/<container_id>/<container_id>-json.log
```

### journald 驱动
```bash
# 使用 journald 驱动
docker run -d \
  --log-driver journald \
  --log-opt tag="{{.ImageName}}/{{.Name}}/{{.ID}}" \
  --log-opt labels=environment \
  nginx:alpine

# 查看 journald 日志
sudo journalctl -u docker CONTAINER_NAME=container_name
sudo journalctl -u docker -f
```

### syslog 驱动
```bash
# 配置 syslog 输出
docker run -d \
  --log-driver syslog \
  --log-opt syslog-address=udp://192.168.1.100:514 \
  --log-opt syslog-format=rfc5424 \
  --log-opt tag="{{.ImageName}}" \
  --log-opt syslog-facility=local3 \
  nginx:alpine
```

### gelf 驱动（Graylog）
```bash
# Graylog 集成
docker run -d \
  --log-driver gelf \
  --log-opt gelf-address=udp://graylog-server:12201 \
  --log-opt tag=nginx-app \
  --log-opt labels=environment \
  --log-opt env=ENVIRONMENT \
  nginx:alpine
```

### fluentd 驱动
```bash
# Fluentd 收集器
docker run -d \
  --log-driver fluentd \
  --log-opt fluentd-address=fluentd-server:24224 \
  --log-opt tag=docker.{{.Name}} \
  --log-opt fluentd-async-connect=true \
  --log-opt labels=app,env \
  nginx:alpine
```

## 日志收集架构

### ELK Stack 集成
```yaml
# docker-compose-elk.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
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

### Filebeat 配置
```yaml
# filebeat.yml
filebeat.inputs:
- type: container
  paths:
    - '/var/lib/docker/containers/*/*.log'

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  indices:
    - index: "docker-logs-%{+yyyy.MM.dd}"
```

## 日志轮转策略

### 自动轮转配置
```bash
# 基于大小的轮转
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=5 \
  --log-opt compress=true \
  nginx:alpine

# 基于时间的轮转（使用外部工具）
docker run -d \
  --log-driver json-file \
  nginx:alpine
```

### 手动日志管理
```bash
# 清理容器日志
docker logs --tail=1000 container_name > backup.log
docker exec container_name truncate -s 0 /var/log/nginx/access.log

# 查找大日志文件
find /var/lib/docker/containers -name "*.log" -size +100M

# 手动轮转日志
sudo truncate -s 0 /var/lib/docker/containers/*/*-json.log
```

## 日志监控与分析

### 实时日志监控
```bash
# 监控所有容器日志
docker logs -f $(docker ps -q)

# 按服务分组监控
docker-compose logs -f

# 过滤特定日志
docker logs container_name 2>&1 | grep "ERROR"

# 监控日志频率
docker logs container_name | awk '{print $5}' | sort | uniq -c | sort -nr
```

### 日志分析脚本
```bash
#!/bin/bash
# 日志分析脚本
LOG_DIR="/var/lib/docker/containers"
OUTPUT_FILE="/tmp/log_analysis_$(date +%Y%m%d).txt"

echo "=== Docker Log Analysis Report ===" > $OUTPUT_FILE
echo "Generated: $(date)" >> $OUTPUT_FILE
echo "" >> $OUTPUT_FILE

# 分析错误日志
echo "=== ERROR Logs ===" >> $OUTPUT_FILE
find $LOG_DIR -name "*.log" -exec grep -l "ERROR" {} \; | \
xargs -I {} grep "ERROR" {} | head -20 >> $OUTPUT_FILE

# 统计日志大小
echo "" >> $OUTPUT_FILE
echo "=== Log File Sizes ===" >> $OUTPUT_FILE
find $LOG_DIR -name "*.log" -exec du -h {} \; | sort -hr >> $OUTPUT_FILE

# 最近修改的日志
echo "" >> $OUTPUT_FILE
echo "=== Recently Modified Logs ===" >> $OUTPUT_FILE
find $LOG_DIR -name "*.log" -exec ls -la {} \; | sort -k8,8 -r | head -10 >> $OUTPUT_FILE
```

## 安全日志管理

### 敏感信息过滤
```bash
# 运行时不记录敏感信息
docker run -d \
  --log-opt env=!API_KEY,!PASSWORD \
  --log-opt labels=!secret \
  nginx:alpine

# 日志脱敏处理
docker logs container_name | sed 's/\(password\|token\)=[^&]*/\1=***/g'
```

### 审计日志配置
```bash
# 启用 Docker 审计日志
sudo mkdir -p /etc/docker/audit/
echo '{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "10"
  }
}' | sudo tee /etc/docker/audit/daemon.json

# 监控 Docker API 调用
sudo auditctl -w /var/run/docker.sock -p rwxa -k docker
```

## 性能优化

### 日志性能调优
```bash
# 异步日志写入
docker run -d \
  --log-driver json-file \
  --log-opt mode=non-blocking \
  --log-opt max-buffer-size=4m \
  nginx:alpine

# 减少日志输出
docker run -d \
  --log-driver json-file \
  --log-opt max-size=1m \
  --log-opt max-file=1 \
  nginx:alpine

# 使用高性能驱动
docker run -d \
  --log-driver journald \
  nginx:alpine
```

### 日志存储优化
```bash
# 使用 tmpfs 存储日志
docker run -d \
  --tmpfs /var/log/nginx \
  nginx:alpine

# 外部日志存储
docker run -d \
  -v /mnt/ssd/logs:/var/log/nginx \
  nginx:alpine
```

## 故障排查

### 日志问题诊断
```bash
# 检查日志驱动状态
docker inspect -f '{{.HostConfig.LogConfig}}' container_name

# 查看日志配置
docker info | grep -A 10 Logging

# 调试日志驱动
docker run --log-driver json-file --log-opt debug=true alpine echo "test"

# 检查日志文件权限
ls -la /var/lib/docker/containers/*/*.log
```

### 常见问题解决
```bash
# 日志文件过大
docker logs --tail=1000 container_name > backup.log
truncate -s 0 $(docker inspect -f '{{.LogPath}}' container_name)

# 日志驱动不工作
docker run --rm --log-driver none alpine echo "test no logs"

# 日志丢失问题
docker run --log-driver json-file --log-opt max-size=10k alpine seq 1 1000
```

## 自动化日志管理

### 日志清理脚本
```bash
#!/bin/bash
# 自动日志清理脚本
LOG_DIR="/var/lib/docker/containers"
MAX_SIZE="100M"
RETENTION_DAYS=30

# 清理过大日志文件
find $LOG_DIR -name "*.log" -size +$MAX_SIZE -exec truncate -s 0 {} \;

# 删除旧日志文件
find $LOG_DIR -name "*.log" -mtime +$RETENTION_DAYS -delete

# 清理空日志文件
find $LOG_DIR -name "*.log" -size 0 -delete
```

### 日志备份脚本
```bash
#!/bin/bash
# 日志备份脚本
BACKUP_DIR="/backup/docker-logs/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# 备份所有容器日志
docker ps -aq | while read container_id; do
  container_name=$(docker inspect -f '{{.Name}}' $container_id | sed 's/\///')
  docker logs --tail=1000 $container_id > $BACKUP_DIR/${container_name}.log
done

# 压缩备份文件
tar czf $BACKUP_DIR.tar.gz $BACKUP_DIR
rm -rf $BACKUP_DIR
```

> 提示：根据业务需求选择合适的日志驱动和配置，生产环境建议使用集中式日志管理系统。

!> 重要：定期监控日志存储使用情况，避免日志文件占满磁盘空间影响系统运行。