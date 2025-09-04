# Docker 性能优化完全指南

## 优化体系架构

```
+----------------+     +----------------+     +----------------+
|   主机层优化   | --> |   容器层优化   | --> |   应用层优化   |
| - 内核参数    |     | - 资源限制    |     | - 代码优化    |
| - 存储驱动    |     | - 镜像优化    |     | - 配置调优    |
| - 网络栈      |     | - 启动参数    |     | - 缓存策略    |
+----------------+     +----------------+     +----------------+
```

## 主机层优化

### 内核参数调优
```bash
# 调整内核参数
echo 'net.core.somaxconn=65535' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog=65535' >> /etc/sysctl.conf
echo 'vm.swappiness=10' >> /etc/sysctl.conf
echo 'vm.overcommit_memory=1' >> /etc/sysctl.conf
sysctl -p

# 文件系统优化
echo 'none /sys/fs/cgroup cgroup defaults 0 0' >> /etc/fstab
mount -o remount,noatime /var/lib/docker
```

### 存储驱动优化
```bash
# 使用 overlay2 驱动
cat > /etc/docker/daemon.json << EOF
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true",
    "overlay2.size=20G"
  ]
}
EOF

# 验证存储驱动
docker info | grep "Storage Driver"
```

### I/O 调度优化
```bash
# 使用高性能I/O调度器
echo 'deadline' > /sys/block/sda/queue/scheduler

# 调整I/O参数
echo '1024' > /sys/block/sda/queue/nr_requests
echo '256' > /sys/block/sda/queue/read_ahead_kb
```

## 容器资源优化

### 内存优化配置
```bash
# 内存限制与交换空间
docker run -d \
  --memory 2g \                 # 内存限制
  --memory-swap 3g \            # 交换空间总量
  --memory-reservation 1g \     # 内存软限制
  --memory-swappiness 10 \     # 交换倾向性（0-100）
  nginx:alpine

# 检查内存使用
docker stats --format "table {{.Name}}\t{{.MemUsage}}\t{{.MemPerc}}"
```

### CPU 优化配置
```bash
# CPU 资源分配
docker run -d \
  --cpus 2.5 \                  # CPU核心数
  --cpuset-cpus "0-3" \         # 绑定CPU核心
  --cpu-shares 512 \            # CPU权重（相对值）
  --cpu-quota 50000 \           # CPU时间片限制（每100ms）
  --cpu-period 100000 \         # CPU周期（微秒）
  nginx:alpine

# 监控CPU使用
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.PIDs}}"
```

### 磁盘 I/O 优化
```bash
# I/O 限制配置
docker run -d \
  --device-read-bps /dev/sda:10mb \     # 读带宽限制
  --device-write-bps /dev/sda:10mb \    # 写带宽限制
  --device-read-iops /dev/sda:1000 \     # 读IOPS限制
  --device-write-iops /dev/sda:1000 \    # 写IOPS限制
  --blkio-weight 500 \                  # 块IO权重（10-1000）
  nginx:alpine
```

## 镜像优化策略

### 多阶段构建优化
```dockerfile
# 构建阶段
FROM golang:1.18 AS builder
WORKDIR /app
COPY . .
RUN go build -ldflags="-s -w" -o app .

# 运行阶段
FROM gcr.io/distroless/base
COPY --from=builder /app/app /
CMD ["/app"]

# 使用UPX压缩（可选）
RUN apt-get update && apt-get install -y upx && \
    upx --best --lzma /app
```

### 层缓存优化
```dockerfile
# 优化层缓存顺序
FROM node:16-alpine

# 先复制package.json安装依赖
COPY package*.json ./
RUN npm install --production

# 然后复制源代码
COPY . .

# 最后构建应用
RUN npm run build

# 清理缓存
RUN npm cache clean --force && \
    rm -rf /tmp/* /var/tmp/*
```

### 镜像最小化
```bash
# 使用小型基础镜像
FROM alpine:3.15
FROM gcr.io/distroless/base
FROM scratch

# 删除不必要的文件
RUN apt-get purge -y curl && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

# 压缩二进制文件
RUN strip /app && upx /app
```

## 网络性能优化

### 网络模式选择
```bash
# 性能对比：host > bridge > overlay
docker run -d --network host nginx:alpine  # 最高性能

# 自定义网络优化
docker network create \
  --driver bridge \
  --opt com.docker.network.bridge.name=br0 \
  --opt com.docker.network.bridge.enable_icc=true \
  mynet
```

### 网络参数调优
```bash
# 调整容器网络参数
docker run -d \
  --sysctl net.core.somaxconn=65535 \
  --sysctl net.ipv4.tcp_tw_reuse=1 \
  --sysctl net.ipv4.tcp_fin_timeout=30 \
  nginx:alpine
```

## 存储性能优化

### 卷性能优化
```bash
# 使用高性能存储卷
docker volume create \
  --driver local \
  --opt type=tmpfs \
  --opt device=tmpfs \
  --opt o=size=100m \
  fast-volume

# 绑定挂载优化
docker run -d \
  -v /mnt/ssd/data:/data:rw,noatime \
  nginx:alpine
```

### 文件系统缓存
```bash
# 使用内存缓存
docker run -d \
  --tmpfs /tmp:size=100m,mode=1777 \
  --tmpfs /cache:size=50m,mode=755 \
  nginx:alpine

# 挂载内存文件系统
docker run -d \
  -v /dev/shm:/dev/shm:rw \
  nginx:alpine
```

## 启动性能优化

### 容器启动优化
```bash
# 优化启动参数
docker run -d \
  --init \                      # 使用init进程
  --ulimit nofile=65536:65536 \ # 文件描述符限制
  --read-only \                 # 只读根文件系统
  nginx:alpine

# 预热容器
docker start container_name
docker stop container_name
```

### 镜像预热
```bash
# 预先拉取镜像
docker pull nginx:alpine
docker pull redis:6.2

# 镜像缓存策略
docker build --cache-from=myapp:latest -t myapp:new .
```

## 监控与调优

### 性能监控工具
```bash
# 实时监控
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemPerc}}\t{{.NetIO}}\t{{.BlockIO}}"

# 使用cAdvisor
docker run -d \
  --name=cadvisor \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -p 8080:8080 \
  google/cadvisor:latest

# 性能分析
docker exec container_name perf top
docker exec container_name strace -c -p 1
```

### 性能基准测试
```bash
# CPU性能测试
docker run --rm --cpus=2 alpine dd if=/dev/zero of=/dev/null bs=1M count=1000

# 磁盘I/O测试
docker run --rm -v /tmp:/test alpine dd if=/dev/zero of=/test/test.bin bs=1M count=100

# 网络性能测试
docker run --rm --network host alpine ping -c 10 8.8.8.8
```

## 应用层优化

### 运行时优化
```bash
# JVM 容器优化
docker run -d \
  -e JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75" \
  openjdk:11-jre

# Node.js 容器优化
docker run -d \
  -e NODE_OPTIONS="--max-old-space-size=4096" \
  node:16-alpine

# Python 容器优化
docker run -d \
  -e PYTHONUNBUFFERED=1 \
  -e PYTHONDONTWRITEBYTECODE=1 \
  python:3.9-alpine
```

### 配置调优
```bash
# Nginx 性能调优
docker run -d \
  -e NGINX_WORKER_PROCESSES=auto \
  -e NGINX_WORKER_CONNECTIONS=1024 \
  nginx:alpine

# Redis 性能调优
docker run -d \
  --sysctl net.core.somaxconn=65535 \
  redis:6.2 --maxclients 10000
```

## 自动化优化脚本

### 性能检查脚本
```bash
#!/bin/bash
# Docker性能检查脚本
echo "=== Docker Performance Check ==="

# 检查资源使用
echo "CPU Usage:"
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}"

echo -e "\nMemory Usage:"
docker stats --no-stream --format "table {{.Name}}\t{{.MemPerc}}"

# 检查镜像大小
echo -e "\nImage Sizes:"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# 检查卷使用
echo -e "\nVolume Usage:"
docker system df -v
```

### 自动化优化脚本
```bash
#!/bin/bash
# 自动优化脚本
echo "Starting Docker optimization..."

# 清理无用资源
docker system prune -af
docker volume prune -f

# 优化镜像
docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "none" | \
while read image; do
    echo "Optimizing $image"
    docker pull $image
done

# 重启容器应用优化
docker restart $(docker ps -q)
```

## 最佳实践总结

### 优化检查清单
1. ✅ 使用适当的基础镜像（Alpine/Distroless）
2. ✅ 实施多阶段构建减少镜像大小
3. ✅ 配置合理的资源限制（CPU/内存/I/O）
4. ✅ 使用高性能网络模式（host/bridge）
5. ✅ 优化存储卷配置（tmpfs/SSD）
6. ✅ 启用容器健康检查和监控
7. ✅ 定期清理无用资源和镜像
8. ✅ 使用最新的Docker引擎和内核

### 性能指标目标
- **启动时间**: < 2秒
- **镜像大小**: < 100MB
- **内存使用**: < 容器限制的80%
- **CPU使用**: < 容器限制的70%
- **网络延迟**: < 1ms（同一主机）
- **磁盘I/O**: 根据业务需求优化

> 提示：性能优化应该基于实际监控数据进行，避免过度优化。

!> 重要：生产环境的性能优化变更应该先在测试环境验证，逐步实施。