# Docker 数据持久化完全指南

## 数据持久化的重要性

容器本身是临时的，当容器被删除时，其文件系统的所有更改都会丢失。数据持久化确保重要数据在容器生命周期之外得以保留。

## 持久化方案对比

| 方案 | 描述 | 适用场景 | 优点 | 缺点 |
|------|------|---------|------|------|
| **Volumes** | Docker 管理的存储卷 | 生产环境、数据库 | 高性能、易备份 | Docker 依赖 |
| **Bind Mounts** | 主机目录挂载 | 开发环境、配置文件 | 直接访问、调试方便 | 主机路径依赖 |
| **tmpfs** | 内存文件系统 | 临时数据、缓存 | 极高速度 | 非持久化 |
| **Cloud Storage** | 云存储集成 | 云环境、分布式 | 可扩展、高可用 | 网络依赖 |

## 数据卷（Volumes）深度实践

### 卷的生命周期管理

#### 创建与配置
```bash
# 创建数据卷
docker volume create app-data

# 创建带标签的卷
docker volume create --label env=production --label app=mysql mysql-data

# 配置卷驱动选项
docker volume create \
  --driver local \
  --opt type=ext4 \
  --opt device=/dev/sdb1 \
  --opt o=defaults \
  custom-volume
```

#### 高级卷操作
```bash
# 批量操作卷
docker volume ls -q | grep "prod-" | xargs docker volume inspect

# 卷数据迁移
docker run --rm \
  -v old-volume:/from \
  -v new-volume:/to \
  alpine rsync -av /from/ /to/

# 卷容量监控
docker system df -v
```

### 生产环境卷配置

#### 数据库卷示例
```bash
# MySQL 数据持久化
docker run -d \
  --name mysql-prod \
  -v mysql_data:/var/lib/mysql \
  -v mysql_config:/etc/mysql/conf.d \
  -v mysql_logs:/var/log/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0 \
  --innodb_buffer_pool_size=1G \
  --innodb_log_file_size=256M
```

#### 应用数据卷示例
```bash
# 多卷配置应用
docker run -d \
  --name myapp \
  -v app_data:/app/data \          # 应用数据
  -v app_logs:/app/logs \          # 日志文件
  -v app_temp:/tmp \              # 临时文件
  -v app_cache:/var/cache \       # 缓存数据
  myapp:latest
```

## 绑定挂载（Bind Mounts）高级用法

### 开发环境配置
```bash
# 开发环境挂载
docker run -d \
  --name dev-app \
  -v $(pwd)/src:/app/src \        # 源代码
  -v $(pwd)/config:/app/config \  # 配置文件
  -v $(pwd)/logs:/app/logs \      # 日志目录
  -v /var/run/docker.sock:/var/run/docker.sock \ # Docker socket
  dev-app:latest
```

### 配置文件管理
```bash
# 配置文件挂载最佳实践
docker run -d \
  --name web-server \
  -v ./nginx.conf:/etc/nginx/nginx.conf:ro \          # 主配置
  -v ./sites-enabled/:/etc/nginx/sites-enabled:ro \  # 站点配置
  -v ./ssl/:/etc/nginx/ssl:ro \                      # SSL证书
  -v ./logs/:/var/log/nginx \                        # 日志目录
  nginx:alpine
```

## 分布式存储集成

### NFS 存储
```bash
# 使用 NFS 卷
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt device=:/nfs/share \
  --opt o=addr=192.168.1.100,rw,soft,timeo=300,retrans=3 \
  nfs-volume

# 运行使用 NFS 的容器
docker run -d \
  -v nfs-volume:/app/shared-data \
  myapp:latest
```

### 云存储集成
```yaml
# docker-compose.yml with cloud storage
version: '3.8'

services:
  app:
    image: myapp:latest
    volumes:
      - aws_volume:/app/data

volumes:
  aws_volume:
    driver: cloudstor:aws
    driver_opts:
      size: "100"
      iops: "1000"
      ebs_type: "gp3"
```

## 数据备份与恢复策略

### 自动化备份方案
```bash
#!/bin/bash
# 数据卷备份脚本
BACKUP_DIR="/backup/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# 备份所有数据卷
docker volume ls -q | while read volume; do
  echo "Backing up volume: $volume"
  docker run --rm \
    -v $volume:/source \
    -v $BACKUP_DIR:/backup \
    alpine \
    sh -c "tar czf /backup/${volume}.tar.gz -C /source . && \
          md5sum /backup/${volume}.tar.gz > /backup/${volume}.md5"
done

# 上传到云存储
aws s3 sync $BACKUP_DIR s3://my-backup-bucket/
```

### 恢复流程
```bash
#!/bin/bash
# 数据卷恢复脚本
RESTORE_DIR="/restore/latest"

# 从云存储下载备份
aws s3 sync s3://my-backup-bucket/ $RESTORE_DIR/

# 恢复数据卷
find $RESTORE_DIR -name "*.tar.gz" | while read backup_file; do
  volume_name=$(basename $backup_file .tar.gz)
  
  # 创建卷（如果不存在）
  docker volume create $volume_name 2>/dev/null || true
  
  # 恢复数据
  docker run --rm \
    -v $volume_name:/target \
    -v $RESTORE_DIR:/backup \
    alpine \
    sh -c "rm -rf /target/* && \
          tar xzf /backup/${volume_name}.tar.gz -C /target && \
          chmod -R 755 /target"
done
```

## 性能优化策略

### 存储性能调优
```bash
# 使用高性能存储驱动
docker run -d \
  --storage-opt dm.basesize=20G \
  --storage-opt dm.blkdiscard=true \
  -v fast_volume:/data \
  myapp:latest

# I/O 调度优化
docker run -d \
  --device-read-bps /dev/sda:10mb \
  --device-write-bps /dev/sda:10mb \
  --device-read-iops /dev/sda:1000 \
  --device-write-iops /dev/sda:1000 \
  -v optimized_volume:/data \
  myapp:latest
```

### 缓存策略
```bash
# 多级缓存配置
docker run -d \
  -v app_data:/app/data \          # 持久化数据
  --tmpfs /app/cache:size=100m \   # 内存缓存
  --mount type=tmpfs,destination=/tmp,tmpfs-size=50m \
  myapp:latest
```

## 安全持久化实践

### 数据加密
```bash
# 使用加密卷
docker volume create \
  --driver local \
  --opt type=crypt \
  --opt keyfile=/path/to/keyfile \
  --opt cipher=aes-xts-plain64 \
  encrypted-volume

# 运行加密容器
docker run -d \
  -v encrypted-volume:/app/secure-data \
  -e ENCRYPTION_KEY=$(cat /path/to/keyfile) \
  secure-app:latest
```

### 权限控制
```bash
# 严格的权限配置
docker run -d \
  -v app_data:/app/data \
  -u 1000:1000 \                  # 非root用户
  --read-only \                   # 只读根文件系统
  --tmpfs /tmp:rw,noexec,nodev \  # 安全临时目录
  myapp:latest
```

## 监控与告警

### 存储监控
```bash
# 监控卷使用情况
docker volume ls -q | xargs -I {} sh -c \
  'echo "Volume: {}"; docker run --rm -v {}:/data alpine df -h /data'

# 设置存储告警
#!/bin/bash
THRESHOLD=80
docker volume ls -q | while read volume; do
  usage=$(docker run --rm -v $volume:/data alpine df /data | awk 'NR==2 {print $5}' | sed 's/%//')
  if [ $usage -gt $THRESHOLD ]; then
    echo "ALERT: Volume $volume usage is $usage%"
    # 发送告警通知
  fi
done
```

## 灾难恢复方案

### 多区域备份
```yaml
# 多区域存储配置
version: '3.8'

services:
  app:
    image: myapp:latest
    volumes:
      - primary_volume:/app/data
      - replica_volume:/app/replica

volumes:
  primary_volume:
    driver: local
    driver_opts:
      type: nfs
      device: ":/primary/data"

  replica_volume:
    driver: local
    driver_opts:
      type: nfs
      device: ":/replica/data"
```

### 数据同步策略
```bash
# 实时数据同步
docker run -d \
  --name sync-agent \
  -v primary_volume:/primary \
  -v replica_volume:/replica \
  alpine \
  sh -c "while true; do rsync -av --delete /primary/ /replica/; sleep 60; done"
```

## 最佳实践总结

1. **数据分类**：根据数据类型选择合适的持久化方案
2. **定期备份**：建立自动化的备份和恢复流程
3. **性能监控**：监控存储性能和使用情况
4. **安全加密**：对敏感数据进行加密存储
5. **权限控制**：使用最小权限原则
6. **测试恢复**：定期测试数据恢复流程

> 提示：根据业务需求选择合适的持久化策略，生产环境建议使用 Volumes 配合自动化备份方案。

!> 重要：确保备份数据的完整性和可恢复性，定期进行恢复测试验证备份有效性。