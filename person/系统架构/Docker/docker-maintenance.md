# Docker 维护管理完全指南

## 系统资源管理

### 资源监控命令
```bash
# 查看 Docker 系统资源使用
docker system df

# 详细资源使用情况
docker system df -v

# 实时监控容器资源
docker stats

# 监控特定容器
docker stats container1 container2

# 格式化输出资源信息
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"
```

### 资源清理操作
```bash
# 清理所有未使用的资源
docker system prune

# 强制清理（包括未停止的容器）
docker system prune -af

# 仅清理镜像
docker image prune

# 清理卷（谨慎使用）
docker volume prune

# 清理网络
docker network prune

# 清理构建缓存
docker builder prune
```

## 容器生命周期管理

### 批量容器操作
```bash
# 停止所有运行中的容器
docker stop $(docker ps -q)

# 删除所有停止的容器
docker rm $(docker ps -aq)

# 重启所有容器
docker restart $(docker ps -q)

# 批量更新容器
docker ps -q | xargs -I {} docker update {} --memory=2g

# 批量设置容器重启策略
docker update --restart=unless-stopped $(docker ps -q)
```

### 容器状态管理
```bash
# 查看容器详细配置
docker inspect container_name

# 查看容器日志
docker logs container_name

# 实时跟踪日志
docker logs -f container_name

# 查看最后 N 行日志
docker logs --tail=100 container_name

# 查看容器进程
docker top container_name

# 查看容器资源限制
docker inspect -f '{{.HostConfig.Memory}}' container_name
```

## 镜像管理

### 镜像维护操作
```bash
# 列出所有镜像
docker images

# 列出镜像并显示大小
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# 删除悬空镜像（无标签）
docker image prune

# 删除指定模式的镜像
docker images | grep "none" | awk '{print $3}' | xargs docker rmi

# 批量删除旧镜像
docker images --filter "dangling=true" -q | xargs docker rmi

# 导出镜像列表
docker images --format "{{.Repository}}:{{.Tag}}" > images.txt
```

### 镜像优化策略
```bash
# 检查镜像层大小
docker history image_name

# 分析镜像内容
docker inspect image_name

# 扫描镜像安全漏洞
docker scan image_name

# 导出镜像用于分析
docker save image_name -o image.tar
```

## 存储管理

### 存储驱动优化
```bash
# 检查当前存储驱动
docker info | grep "Storage Driver"

# Overlay2 驱动配置
cat /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true",
    "overlay2.size=20G"
  ]
}
```

### 卷管理最佳实践
```bash
# 查看卷使用情况
docker volume ls
docker volume inspect volume_name

# 备份数据卷
docker run --rm -v source_volume:/source -v /backup:/backup alpine tar czf /backup/backup.tar.gz -C /source .

# 迁移卷数据
docker run --rm -v old_volume:/from -v new_volume:/to alpine cp -a /from/. /to/
```

## 网络管理

### 网络维护操作
```bash
# 查看网络列表
docker network ls

# 查看网络详情
docker network inspect network_name

# 清理未使用的网络
docker network prune

# 检查网络连通性
docker exec container_name ping other_container

# 查看容器网络配置
docker exec container_name ip addr show
```

### 网络故障排查
```bash
# 检查 DNS 解析
docker exec container_name nslookup google.com

# 查看网络端口映射
docker port container_name

# 检查网络流量
docker exec container_name netstat -tuln

# 测试网络性能
docker exec container_name ping -c 4 other_container
```

## 日志管理

### 日志配置与管理
```bash
# 配置日志驱动（daemon.json）
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}

# 查看日志驱动配置
docker info --format '{{.LoggingDriver}}'

# 实时收集所有容器日志
docker logs -f $(docker ps -q)

# 导出容器日志
docker logs container_name > container.log

# 使用日志轮转工具
docker run -d --name logspout \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gliderlabs/logspout
```

## 安全维护

### 安全最佳实践
```bash
# 定期更新 Docker 引擎
sudo apt-get update && sudo apt-get upgrade docker-ce

# 扫描镜像漏洞
docker scan image_name

# 检查容器安全配置
docker inspect --format='{{.HostConfig.Privileged}}' container_name

# 检查容器能力
docker inspect --format='{{.HostConfig.CapAdd}}' container_name

# 使用非 root 用户运行容器
docker run -u 1000:1000 image_name
```

### 安全审计命令
```bash
# 检查运行中的容器安全状态
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# 检查容器权限提升
docker inspect -f '{{.HostConfig.Privileged}}' container_name

# 检查挂载点安全
docker inspect -f '{{.Mounts}}' container_name

# 检查环境变量安全性
docker exec container_name env | grep -i password
```

## 备份与恢复

### 完整备份策略
```bash
#!/bin/bash
# 完整 Docker 环境备份脚本
BACKUP_DIR="/backup/docker/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# 备份所有容器配置
docker ps -aq | xargs -I {} sh -c \
  'docker inspect {} > "$0/container_{}.json"' $BACKUP_DIR

# 备份所有镜像
docker save $(docker images -q) -o $BACKUP_DIR/images.tar

# 备份所有卷数据
docker volume ls -q | xargs -I {} sh -c \
  'docker run --rm -v {}:/data -v $0:/backup alpine tar czf /backup/volume_{}.tar.gz -C /data .' $BACKUP_DIR

# 备份 Docker 配置
cp /etc/docker/daemon.json $BACKUP_DIR/
```

### 恢复流程
```bash
#!/bin/bash
# Docker 环境恢复脚本
RESTORE_DIR="/backup/docker/20230101_120000"

# 恢复镜像
docker load -i $RESTORE_DIR/images.tar

# 恢复卷数据
for volume_file in $RESTORE_DIR/volume_*.tar.gz; do
  volume_name=$(basename $volume_file .tar.gz | sed 's/volume_//')
  docker volume create $volume_name
  docker run --rm -v $volume_name:/data -v $RESTORE_DIR:/backup alpine \
    tar xzf /backup/volume_${volume_name}.tar.gz -C /data
done
```

## 性能监控

### 监控工具集成
```bash
# 使用 cAdvisor 监控
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  google/cadvisor:latest

# 使用 Prometheus 监控
docker run -d \
  --name=prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# 使用 Grafana 可视化
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  grafana/grafana
```

### 性能调优命令
```bash
# 检查容器性能瓶颈
docker stats --no-stream

# 分析容器 I/O
docker exec container_name iostat -x 1

# 检查内存使用
docker exec container_name free -h

# 监控网络流量
docker exec container_name iftop

# 分析系统调用
docker exec container_name strace -p 1
```

## 自动化维护

### 定时维护脚本
```bash
#!/bin/bash
# 每日维护脚本
echo "$(date): Starting Docker maintenance"

# 清理资源
docker system prune -f

# 更新镜像
docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "none" | \
  xargs -I {} docker pull {}

# 重启容器（优雅方式）
docker ps -q | xargs -I {} docker restart {}

# 检查健康状态
docker ps --format "table {{.Names}}\t{{.Status}}"

echo "$(date): Maintenance completed"
```

### 监控告警配置
```bash
#!/bin/bash
# 容器状态监控脚本
THRESHOLD_CPU=80
THRESHOLD_MEM=90

docker stats --no-stream --format \
  "{{.Name}} {{.CPUPerc}} {{.MemPerc}}" | \
while read name cpu mem; do
  cpu_num=$(echo $cpu | sed 's/%//')
  mem_num=$(echo $mem | sed 's/%//')
  
  if [ $(echo "$cpu_num > $THRESHOLD_CPU" | bc) -eq 1 ]; then
    echo "ALERT: Container $name CPU usage: $cpu"
  fi
  
  if [ $(echo "$mem_num > $THRESHOLD_MEM" | bc) -eq 1 ]; then
    echo "ALERT: Container $name Memory usage: $mem"
  fi
done
```

## 故障排查指南

### 常见问题解决
```bash
# Docker 服务状态检查
sudo systemctl status docker
sudo journalctl -u docker.service

# 容器启动失败排查
docker logs container_name

# 网络问题诊断
docker network inspect bridge
iptables -L -n -t nat

# 存储问题排查
docker info | grep -A 10 "Storage"
df -h /var/lib/docker

# 权限问题解决
sudo chmod 666 /var/run/docker.sock
```

### 调试模式启用
```bash
# 启用 Docker 调试日志
sudo mkdir -p /etc/systemd/system/docker.service.d/
echo '[Service]
Environment="DOCKER_OPTS=--debug"' | sudo tee /etc/systemd/system/docker.service.d/debug.conf

sudo systemctl daemon-reload
sudo systemctl restart docker

# 查看调试日志
sudo journalctl -u docker.service -f
```

> 提示：定期执行维护任务可以保持 Docker 环境的高效和稳定运行。

!> 重要：生产环境执行清理操作前务必进行备份，避免误删重要数据。