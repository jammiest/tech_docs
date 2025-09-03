# Docker 存储卷管理完全指南

## 概述

Docker 存储卷（Volumes）是持久化 Docker 容器生成和使用的数据的首选机制。与容器生命周期解耦，数据可以在容器之间共享和重用。

## 存储类型对比

| 存储类型 | 描述 | 适用场景 | 性能 | 持久性 |
|---------|------|---------|------|--------|
| **Volumes** | Docker 管理的数据卷 | 生产环境数据持久化 | 高 | 高 |
| **Bind Mounts** | 主机文件系统挂载 | 开发环境、配置文件 | 中 | 依赖主机 |
| **tmpfs** | 内存文件系统 | 临时数据、敏感信息 | 极高 | 无 |
| **Named Pipes** | Windows 命名管道 | Windows 容器通信 | - | - |

## 数据卷（Volumes）

### 卷操作命令

#### 基础管理
```bash
# 创建数据卷
docker volume create my-volume

# 查看所有数据卷
docker volume ls

# 查看卷详情
docker volume inspect my-volume

# 删除数据卷
docker volume rm my-volume

# 清理未使用的数据卷
docker volume prune
```

#### 使用数据卷
```bash
# 运行容器时使用数据卷
docker run -d \
  --name mysql \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0

# 使用只读数据卷
docker run -d \
  -v config_volume:/app/config:ro \
  nginx:alpine

# 复制文件到数据卷
docker run --rm \
  -v my_volume:/target \
  alpine cp /path/to/file /target/
```

### 卷驱动配置

#### 本地驱动
```bash
# 创建使用本地驱动的卷
docker volume create --driver local \
  --opt type=nfs \
  --opt device=:/nfs/share \
  --opt o=addr=192.168.1.100,rw \
  nfs_volume
```

#### 第三方驱动
```bash
# 使用第三方卷驱动（需要先安装驱动）
docker volume create --driver vieux/sshfs \
  --opt sshcmd=user@host:/remote/path \
  --opt password=secret \
  ssh_volume
```

## 绑定挂载（Bind Mounts）

### 绑定挂载使用
```bash
# 基本绑定挂载
docker run -d \
  --name web \
  -v /host/path:/container/path \
  nginx:alpine

# 使用相对路径
docker run -d \
  -v ./config:/app/config \
  myapp

# 只读绑定挂载
docker run -d \
  -v /host/config:/app/config:ro \
  myapp

# 递归绑定挂载（子目录也会挂载）
docker run -d \
  -v /host/data:/data:z \
  myapp
```

### 权限配置
```bash
# SELinux 标签配置
docker run -d \
  -v /host/data:/data:z \        # 共享标签
  -v /host/config:/config:Z \    # 私有标签
  myapp

# 用户和组权限
docker run -d \
  -v /host/data:/data \
  -u 1000:1000 \
  myapp
```

## tmpfs 挂载

### 内存文件系统
```bash
# 使用 tmpfs
docker run -d \
  --tmpfs /tmp:size=100m,mode=1777 \
  myapp

# 等效的 mount 语法
docker run -d \
  --mount type=tmpfs,destination=/tmp,tmpfs-size=100m,tmpfs-mode=1777 \
  myapp
```

### tmpfs 配置选项
```bash
# 多个 tmpfs 配置
docker run -d \
  --tmpfs /tmp:size=100m,mode=1777 \
  --tmpfs /cache:size=50m,mode=755 \
  myapp

# 使用 mount 语法进行精细控制
docker run -d \
  --mount type=tmpfs,destination=/tmp,\
tmpfs-size=100m,tmpfs-mode=1777,tmpfs-uid=1000 \
  myapp
```

## Docker Compose 中的存储配置

### 数据卷配置
```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  app:
    image: myapp:latest
    volumes:
      - app_logs:/app/logs
      - ./config:/app/config:ro

volumes:
  db_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /opt/data/postgres

  app_logs:
    driver: local
```

### 高级卷配置
```yaml
volumes:
  # NFS 卷
  nfs_volume:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw,nfsvers=4
      device: ":/exports/data"

  # 加密卷
  encrypted_volume:
    driver: local
    driver_opts:
      type: crypt
      keyfile: /path/to/keyfile

  # 外部卷
  existing_volume:
    external: true
    name: production_data
```

## 存储驱动详解

### Overlay2 驱动（推荐）
```bash
# 检查当前存储驱动
docker info | grep "Storage Driver"

# Overlay2 配置
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true",
    "overlay2.size=20G"
  ]
}
```

### 存储驱动比较
| 驱动 | 优点 | 缺点 | 适用场景 |
|------|------|------|---------|
| **overlay2** | 性能好，内核支持 | 需要较新内核 | 生产环境首选 |
| **aufs** | 成熟稳定 | 未进入主流内核 | 旧系统兼容 |
| **devicemapper** | 企业级特性 | 配置复杂 | 特定企业需求 |
| **btrfs** | 快照功能 | 稳定性问题 | 开发测试环境 |

## 数据管理策略

### 备份与恢复
```bash
# 备份数据卷
docker run --rm \
  -v db_data:/source \
  -v /host/backup:/backup \
  alpine tar czf /backup/db_backup.tar.gz -C /source .

# 恢复数据卷
docker run --rm \
  -v db_data:/target \
  -v /host/backup:/backup \
  alpine tar xzf /backup/db_backup.tar.gz -C /target

# 卷数据迁移
docker run --rm \
  -v old_volume:/from \
  -v new_volume:/to \
  alpine cp -a /from/. /to/
```

### 数据清理
```bash
# 清理容器产生的数据
docker run --rm \
  -v my_volume:/data \
  alpine sh -c 'rm -rf /data/*'

# 查看卷使用情况
docker system df -v

# 安全删除敏感数据
docker run --rm \
  -v sensitive_volume:/data \
  alpine sh -c 'shred -u /data/* && rm -rf /data/*'
```

## 安全最佳实践

### 安全配置
```bash
# 使用只读挂载
docker run -d \
  -v config_volume:/app/config:ro \
  -v /etc/passwd:/etc/passwd:ro \
  myapp

# 避免敏感信息挂载
docker run -d \
  --read-only \
  --tmpfs /tmp \
  myapp

# 使用专用用户
docker run -d \
  -v app_data:/data \
  -u appuser \
  myapp
```

### 审计与监控
```bash
# 监控卷使用情况
docker volume ls -q | xargs docker volume inspect

# 检查卷权限
docker run --rm \
  -v my_volume:/data \
  alpine ls -la /data

# 审计挂载点
docker inspect --format='{{.Mounts}}' container_name
```

## 性能优化

### 存储性能调优
```bash
# 使用高性能存储驱动
docker run --storage-opt dm.basesize=20G

# 调整 I/O 调度
docker run --device-write-bps /dev/sda:1mb

# 使用内存缓存
docker run -d \
  --mount type=tmpfs,destination=/cache \
  myapp
```

### 网络存储优化
```yaml
# docker-compose.yml
volumes:
  nfs_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw,nfsvers=4.1,timeo=300,retrans=3
      device: ":/data"
```

## 故障排查

### 常见问题解决
```bash
# 卷权限问题
docker run --rm -v my_volume:/data alpine chown -R 1000:1000 /data

# 挂载点检查
docker exec container_name ls -la /mount/point

# 存储驱动问题
docker info | grep -A 10 "Storage Driver"

# 卷空间清理
docker system prune --volumes
```

### 调试命令
```bash
# 查看所有挂载点
docker inspect -f '{{json .Mounts}}' container_name | jq .

# 检查卷内容
docker run --rm -v my_volume:/data alpine ls -la /data

# 测试存储性能
docker run --rm -v test_volume:/data alpine dd if=/dev/zero of=/data/test bs=1M count=100
```

## 企业级实践

### 多主机卷管理
```yaml
# 使用 overlay 网络的多主机卷
version: '3.8'

services:
  app:
    image: myapp:latest
    volumes:
      - shared_data:/app/data
    deploy:
      mode: global

volumes:
  shared_data:
    driver: flocker
    driver_opts:
      policy: golden
```

### 备份策略
```bash
#!/bin/bash
# 卷备份脚本
VOLUMES=$(docker volume ls -q)
BACKUP_DIR="/backup/$(date +%Y%m%d)"

mkdir -p $BACKUP_DIR

for volume in $VOLUMES; do
  docker run --rm \
    -v $volume:/source \
    -v $BACKUP_DIR:/backup \
    alpine tar czf /backup/${volume}.tar.gz -C /source .
done
```

> 提示：根据数据重要性选择合适的存储类型，生产环境推荐使用 Docker Volumes 进行数据持久化。

!> 重要：定期备份重要数据卷，并测试恢复流程。避免将敏感数据存储在容器可写层中。