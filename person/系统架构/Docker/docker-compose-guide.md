# Docker Compose 完全指南

## 概述

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。通过一个 YAML 文件配置应用的服务，然后使用单个命令即可创建和启动所有服务。

## 核心概念

### Compose 文件结构
```yaml
version: '3.8'  # Compose 文件版本
services:        # 服务定义
  web:           # 服务名称
    image: nginx # 服务配置
  db:
    image: postgres
networks:        # 网络定义
  app-net:
volumes:         # 卷定义
  db-data:
```

## 安装与配置

### 安装 Docker Compose
```bash
# Linux 安装
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

### 版本兼容性
| Docker Engine | Compose 文件版本 |
|---------------|------------------|
| 19.03.0+      | 3.8              |
| 18.06.0+      | 3.7              |
| 18.02.0+      | 3.6              |
| 17.12.0+      | 3.5              |

## Compose 文件详解

### 服务配置 (Services)

#### 基础配置
```yaml
services:
  web:
    # 镜像配置
    image: nginx:alpine
    # 或使用构建
    build:
      context: .
      dockerfile: Dockerfile
      args:
        BUILD_ENV: production

    # 容器名称
    container_name: my-web-app

    # 重启策略
    restart: unless-stopped

    # 依赖关系
    depends_on:
      - db
      - redis
```

#### 网络配置
```yaml
services:
  web:
    # 端口映射
    ports:
      - "80:80"           # 主机端口:容器端口
      - "443:443"
      - "3000-3005:3000-3005"  # 端口范围

    # 网络配置
    networks:
      - frontend
      - backend

    # DNS 配置
    dns:
      - 8.8.8.8
      - 1.1.1.1
    dns_search:
      - example.com
```

#### 存储配置
```yaml
services:
  web:
    # 卷挂载
    volumes:
      - /host/path:/container/path        # 绑定挂载
      - named_volume:/container/path      # 命名卷
      - ./relative/path:/container/path   # 相对路径
      - config:/app/config:ro            # 只读挂载

    # 临时文件系统
    tmpfs:
      - /tmp:size=100m,mode=1777

  db:
    # 数据持久化
    volumes:
      - db_data:/var/lib/postgresql/data
```

#### 环境配置
```yaml
services:
  app:
    # 环境变量
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379

    # 环境变量文件
    env_file:
      - .env
      - .env.production

    # 工作目录
    working_dir: /app

    # 用户权限
    user: "1000:1000"
```

#### 资源限制
```yaml
services:
  app:
    # 部署配置（Swarm 模式）
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M

    # 传统资源限制
    mem_limit: 512m
    memswap_limit: 1g
    cpus: 0.5
```

### 健康检查
```yaml
services:
  web:
    # 健康检查配置
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
```

### 网络配置 (Networks)
```yaml
networks:
  # 默认网络配置
  default:
    driver: bridge

  # 自定义网络
  frontend:
    driver: bridge
    driver_opts:
      com.docker.network.driver.mtu: 1450
    ipam:
      config:
        - subnet: 172.16.238.0/24
          gateway: 172.16.238.1

  backend:
    driver: overlay
    external: true  # 使用外部网络
```

### 卷配置 (Volumes)
```yaml
volumes:
  # 命名卷
  db_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.1,rw
      device: ":/path/to/nfs/share"

  # 外部卷
  app_data:
    external: true

  # 配置卷
  config:
    driver: local
```

## 完整示例

### Web 应用栈示例
```yaml
version: '3.8'

services:
  # 前端服务
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://backend:8000
    depends_on:
      - backend
    networks:
      - app-network

  # 后端服务
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgres://user:password@db:5432/app
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./backend:/app
      - /app/node_modules
    networks:
      - app-network

  # 数据库服务
  db:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  # Redis 服务
  redis:
    image: redis:6-alpine
    command: redis-server --requirepass password
    volumes:
      - redis_data:/data
    networks:
      - app-network

  # 反向代理
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

## 命令使用

### 基本操作命令
```bash
# 启动服务（后台运行）
docker-compose up -d

# 启动服务并构建镜像
docker-compose up --build

# 停止服务
docker-compose down

# 查看服务状态
docker-compose ps

# 查看服务日志
docker-compose logs
docker-compose logs -f  # 实时日志
docker-compose logs web  # 特定服务日志

# 重启服务
docker-compose restart

# 缩放服务实例数
docker-compose up --scale web=3 --scale worker=2
```

### 服务管理命令
```bash
# 执行命令
docker-compose exec web bash
docker-compose exec db psql -U user app

# 运行一次性命令
docker-compose run --rm web npm test

# 查看服务配置
docker-compose config
docker-compose config --services  # 查看所有服务

# 暂停和恢复服务
docker-compose pause
docker-compose unpause

# 强制重建服务
docker-compose up --force-recreate
```

### 环境管理
```bash
# 使用不同环境文件
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

# 指定环境变量文件
docker-compose --env-file .env.production up

# 查看环境变量
docker-compose config --services | xargs -I {} sh -c 'echo {}; docker-compose run --rm {} env'
```

## 高级特性

### 多文件配置
```bash
# 基础配置
docker-compose -f docker-compose.yml -f docker-compose.override.yml up

# 生产环境配置
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

### 扩展配置示例
```yaml
# docker-compose.override.yml
version: '3.8'

services:
  web:
    volumes:
      - .:/app  # 开发时挂载源代码
    ports:
      - "9229:9229"  # 调试端口

  db:
    ports:
      - "5432:5432"  # 暴露数据库端口用于开发
```

### 变量替换
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: ${IMAGE_NAME:-nginx}:${IMAGE_TAG:-latest}
    ports:
      - "${HOST_PORT:-80}:80"

# .env 文件
IMAGE_NAME=myapp
IMAGE_TAG=1.0.0
HOST_PORT=8080
```

## 最佳实践

### 开发环境配置
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  web:
    build:
      target: development  # 多阶段构建的开发目标
    volumes:
      - .:/app
      - /app/node_modules  # 避免覆盖node_modules
    environment:
      - NODE_ENV=development
    ports:
      - "9229:9229"  # 调试端口
    command: npm run dev  # 开发模式启动
```

### 生产环境配置
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  web:
    build:
      target: production  # 多阶段构建的生产目标
    environment:
      - NODE_ENV=production
    restart: always
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

### 安全配置
```yaml
services:
  app:
    read_only: true  # 只读根文件系统
    tmpfs:
      - /tmp:rw,noexec,nodev,nosuid
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

## 故障排查

### 常见问题解决
```bash
# 查看服务状态
docker-compose ps

# 查看详细日志
docker-compose logs --tail=100 web

# 进入容器调试
docker-compose exec web sh

# 检查网络连通性
docker-compose exec web ping db

# 验证配置文件
docker-compose config

# 强制重建
docker-compose up --build --force-recreate
```

### 调试技巧
```bash
# 详细输出模式
docker-compose --verbose up

# 只验证不执行
docker-compose config --quiet

# 查看服务依赖图
docker-compose config --services
```

## 性能优化

### 构建优化
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      cache_from:
        - myapp:latest
      # 构建参数优化
      args:
        NODE_ENV: production
        NPM_REGISTRY: https://registry.npmjs.org
```

### 资源优化
```yaml
services:
  app:
    # 使用内存限制
    mem_limit: 1g
    memswap_limit: 2g
    
    # CPU限制
    cpus: 2
    cpu_shares: 1024
    
    # I/O限制
    blkio_config:
      weight: 300
```

> 提示：Docker Compose 极大地简化了多容器应用的管理，适合开发、测试和生产环境的容器编排。

!> 重要：生产环境使用时应确保配置适当的安全设置和资源限制，避免安全风险和性能问题。