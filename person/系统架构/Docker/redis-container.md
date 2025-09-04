# Redis 容器化部署完全指南

## 基础部署

### 快速启动 Redis 容器
```bash
# 最新版本 Redis
docker run -d \
  --name redis-server \
  -p 6379:6379 \
  redis:latest

# 指定版本 (如 6.2)
docker run -d \
  --name redis-6.2 \
  -p 6380:6379 \
  redis:6.2
```

### 核心环境变量
| 变量名 | 描述 | 示例值 |
|--------|------|--------|
| `REDIS_PASSWORD` | Redis 访问密码 | mysecretpassword |
| `REDIS_PORT` | 自定义端口 | 6380 |
| `REDIS_APPENDONLY` | 启用 AOF 持久化 | yes |
| `REDIS_DATABASES` | 数据库数量 | 16 |

## 数据持久化配置

### 使用数据卷持久化
```bash
# 创建专用数据卷
docker volume create redis_data

# 运行带持久化的 Redis
docker run -d \
  --name redis-persistent \
  -v redis_data:/data \
  -e REDIS_PASSWORD=password \
  redis:6.2 \
  --appendonly yes
```

### 绑定挂载配置
```bash
# 使用主机目录存储数据
mkdir -p /opt/redis/{data,conf}

# 运行 Redis 并挂载配置目录
docker run -d \
  --name redis-custom \
  -v /opt/redis/data:/data \
  -v /opt/redis/conf:/usr/local/etc/redis \
  -e REDIS_PASSWORD=password \
  redis:6.2 \
  --appendonly yes
```

## 配置文件定制

### 自定义 redis.conf 配置
```bash
# 创建自定义配置文件目录
mkdir -p /opt/redis/conf

# 下载默认配置文件
wget -O /opt/redis/conf/redis.conf https://raw.githubusercontent.com/redis/redis/6.2/redis.conf

# 修改关键配置
sed -i 's/^# requirepass foobared/requirepass mypassword/' /opt/redis/conf/redis.conf
sed -i 's/^appendonly no/appendonly yes/' /opt/redis/conf/redis.conf
sed -i 's/^maxmemory-policy noeviction/maxmemory-policy allkeys-lru/' /opt/redis/conf/redis.conf

# 运行带自定义配置的 Redis
docker run -d \
  --name redis-custom-config \
  -v /opt/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  -v /opt/redis/data:/data \
  redis:6.2 \
  redis-server /usr/local/etc/redis/redis.conf
```

### 常用性能参数
```redis
# 内存管理
maxmemory 2gb
maxmemory-policy allkeys-lru

# 持久化配置
appendonly yes
appendfsync everysec

# 连接配置
maxclients 10000
tcp-backlog 511
timeout 300

# 安全配置
requirepass mysecretpassword
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG ""
```

## 安全配置

### 安全启动选项
```bash
# 使用密码保护
docker run -d \
  --name redis-secure \
  -e REDIS_PASSWORD=complexpassword \
  redis:6.2

# 禁用危险命令
docker run -d \
  --name redis-safe \
  redis:6.2 \
  --requirepass mypassword \
  --rename-command FLUSHDB "" \
  --rename-command FLUSHALL "" \
  --rename-command CONFIG ""
```

### 生产环境安全建议
1. **强制密码认证**：设置复杂密码
2. **禁用危险命令**：如 FLUSHDB, FLUSHALL, CONFIG
3. **网络隔离**：使用专用网络
4. **TLS 加密**：启用 SSL 加密连接
5. **定期备份**：设置 RDB/AOF 备份策略

## 网络与连接

### 网络配置
```bash
# 创建专用网络
docker network create redis-network

# 在专用网络中运行 Redis
docker run -d \
  --name redis-net \
  --network redis-network \
  -e REDIS_PASSWORD=password \
  redis:6.2

# 连接到此网络的应用程序
docker run -d \
  --name myapp \
  --network redis-network \
  -e REDIS_HOST=redis-net \
  -e REDIS_PASSWORD=password \
  myapp:latest
```

### 连接测试
```bash
# 使用 redis-cli 连接
docker exec -it redis-server redis-cli

# 带密码认证连接
docker exec -it redis-server redis-cli -a password

# 从外部主机连接
redis-cli -h 127.0.0.1 -p 6379 -a password

# 使用临时客户端容器连接
docker run -it --rm \
  --network redis-network \
  redis:6.2 \
  redis-cli -h redis-net -a password
```

## 备份与恢复

### 数据备份
```bash
# 手动创建 RDB 快照
docker exec redis-server redis-cli -a password SAVE

# 备份 RDB 文件
docker cp redis-server:/data/dump.rdb /backup/redis-$(date +%Y%m%d).rdb

# 定时备份脚本
docker run -d \
  --name redis-backup \
  --volumes-from redis-server \
  -v /backups:/backups \
  -e REDIS_PASSWORD=password \
  alpine \
  sh -c 'while true; do \
    docker exec redis-server redis-cli -a password SAVE && \
    cp /data/dump.rdb /backups/redis-$(date +%Y%m%d%H%M%S).rdb; \
    sleep 86400; done'
```

### 数据恢复
```bash
# 停止 Redis 服务
docker stop redis-server

# 恢复 RDB 文件
docker cp /backup/redis-20230101.rdb redis-server:/data/dump.rdb

# 启动 Redis
docker start redis-server
```

## 性能优化

### 资源限制
```bash
# 限制容器资源
docker run -d \
  --name redis-optimized \
  --cpus=2 \
  --memory=4g \
  --memory-swap=4g \
  --ulimit nofile=65536:65536 \
  -v redis_data:/data \
  -e REDIS_PASSWORD=password \
  redis:6.2 \
  --maxmemory 3gb \
  --maxmemory-policy allkeys-lru
```

### 生产环境推荐参数
```bash
docker run -d \
  --name redis-production \
  -v redis_data:/data \
  -v /opt/redis/conf:/usr/local/etc/redis \
  -e REDIS_PASSWORD=password \
  redis:6.2 \
  --maxmemory 3gb \
  --maxmemory-policy volatile-lru \
  --appendonly yes \
  --appendfsync everysec \
  --save "" \
  --activerehashing yes \
  --hz 10 \
  --tcp-backlog 511 \
  --client-output-buffer-limit normal 0 0 0 \
  --client-output-buffer-limit replica 256mb 64mb 60 \
  --client-output-buffer-limit pubsub 32mb 8mb 60
```

## 监控与维护

### 监控 Redis 状态
```bash
# 查看 Redis 信息
docker exec redis-server redis-cli -a password INFO

# 查看内存使用
docker exec redis-server redis-cli -a password INFO MEMORY

# 查看客户端连接
docker exec redis-server redis-cli -a password CLIENT LIST

# 查看慢查询
docker exec redis-server redis-cli -a password SLOWLOG GET
```

### 日志管理
```bash
# 查看 Redis 日志
docker logs redis-server

# 实时监控日志
docker logs -f redis-server

# 配置日志级别 (redis.conf)
loglevel notice
logfile /data/redis.log
```

## Docker Compose 配置

### 完整 Redis 服务配置
```yaml
version: '3.8'

services:
  redis:
    image: redis:6.2
    container_name: redis-server
    command: redis-server --requirepass $$REDIS_PASSWORD --appendonly yes
    environment:
      - REDIS_PASSWORD=mysecretpassword
    volumes:
      - redis_data:/data
      - ./conf/redis.conf:/usr/local/etc/redis/redis.conf
    ports:
      - "6379:6379"
    networks:
      - redis-network
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "$$REDIS_PASSWORD", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

  redis-commander:
    image: rediscommander/redis-commander:latest
    ports:
      - "8081:8081"
    environment:
      - REDIS_HOSTS=local:redis-server:6379
      - REDIS_PASSWORD=mysecretpassword
    networks:
      - redis-network
    depends_on:
      - redis

networks:
  redis-network:
    driver: bridge

volumes:
  redis_data:
    driver: local
```

## 高可用方案

### Redis 主从复制
```yaml
# docker-compose-redis-replication.yml
version: '3.8'

services:
  redis-master:
    image: redis:6.2
    command: redis-server --requirepass masterpassword --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_master_data:/data
    networks:
      - redis-cluster

  redis-replica:
    image: redis:6.2
    command: >
      redis-server --replicaof redis-master 6379
      --masterauth masterpassword
      --requirepass replicapassword
      --appendonly yes
    ports:
      - "6380:6379"
    volumes:
      - redis_replica_data:/data
    depends_on:
      - redis-master
    networks:
      - redis-cluster

networks:
  redis-cluster:
    driver: bridge

volumes:
  redis_master_data:
  redis_replica_data:
```

## 故障排查

### 常见问题解决
```bash
# 容器无法启动
docker logs redis-server

# 连接问题检查
docker exec redis-server redis-cli -a password PING

# 内存问题诊断
docker exec redis-server redis-cli -a password INFO MEMORY

# 性能问题诊断
docker exec redis-server redis-cli -a password --latency
docker exec redis-server redis-cli -a password --stat
```

### 维护命令
```bash
# 安全关闭 Redis
docker exec redis-server redis-cli -a password SHUTDOWN SAVE

# 清除所有数据 (危险)
docker exec redis-server redis-cli -a password FLUSHALL

# 重建 AOF 文件
docker exec redis-server redis-cli -a password BGREWRITEAOF
```

> 提示：生产环境部署 Redis 容器时，务必配置适当的数据持久化、资源限制和定期备份策略。

!> 重要：Redis 密码应通过安全方式管理，避免在命令行或脚本中明文存储。