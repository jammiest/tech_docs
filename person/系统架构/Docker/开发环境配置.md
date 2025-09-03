# Docker 开发环境配置完全指南

## 开发环境架构

```
+----------------+     +----------------+     +----------------+
|   本地开发工具  | --> |   Docker环境   | --> |   云服务集成   |
| - IDE插件     |     | - 容器化服务  |     | - 云数据库    |
| - 调试工具    |     | - 热重载      |     | - 消息队列    |
| - 构建工具    |     | - 代码映射    |     | - 对象存储    |
+----------------+     +----------------+     +----------------+
```

## 本地开发工具配置

### IDE 插件配置
```bash
# VS Code Docker 扩展配置
# settings.json
{
  "docker.command": "docker",
  "docker.languageserver": {
    "enabled": true,
    "suggestions": true
  },
  "docker.composeCommand": "docker-compose",
  "docker.showExplorer": true
}

# IntelliJ Docker 集成
# 启用Docker支持，配置Docker守护进程连接
```

### 开发工具安装
```bash
# 安装 Docker Desktop (Mac/Windows)
# https://www.docker.com/products/docker-desktop

# Linux 开发环境
sudo apt-get update
sudo apt-get install docker.io docker-compose

# 开发工具链
sudo apt-get install git curl wget make jq
```

## 项目结构规划

### 标准项目结构
```
my-project/
├── docker/
│   ├── dev/
│   │   ├── Dockerfile
│   │   └── docker-compose.dev.yml
│   └── prod/
│       ├── Dockerfile
│       └── docker-compose.prod.yml
├── src/
│   ├── app/
│   ├── tests/
│   └── requirements.txt
├── .dockerignore
├── docker-compose.yml
└── Makefile
```

### .dockerignore 文件
```dockerignore
# 开发环境 .dockerignore
.git
.gitignore
README.md
*.log
tmp/
node_modules/
.env
.venv/
__pycache__/
*.pyc
*.pyo
.DS_Store
```

## Docker Compose 开发配置

### 开发环境 compose 文件
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  # 应用服务
  web:
    build:
      context: .
      dockerfile: docker/dev/Dockerfile
      target: development
    ports:
      - "3000:3000"
      - "9229:9229"  # Node.js 调试端口
    volumes:
      - .:/app
      - /app/node_modules  # 避免覆盖node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=true
    command: npm run dev
    depends_on:
      - db
      - redis

  # 数据库
  db:
    image: postgres:13
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=developer
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp_dev
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Redis
  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes

  # 开发工具
  adminer:
    image: adminer
    ports:
      - "8080:8080"

volumes:
  postgres_data:
```

## 开发专用 Dockerfile

### 多阶段开发构建
```dockerfile
# docker/dev/Dockerfile
FROM node:16 AS development

# 安装开发工具
RUN apt-get update && apt-get install -y \
    curl \
    git \
    vim \
    && rm -rf /var/lib/apt/lists/*

# 设置工作目录
WORKDIR /app

# 复制package文件
COPY package*.json ./

# 安装依赖（包括开发依赖）
RUN npm install

# 复制源代码
COPY . .

# 暴露端口
EXPOSE 3000
EXPOSE 9229  # 调试端口

# 开发模式启动
CMD ["npm", "run", "dev"]
```

### Python 开发环境
```dockerfile
# docker/dev/Dockerfile.python
FROM python:3.9-slim AS development

# 安装开发工具
RUN apt-get update && apt-get install -y \
    git \
    curl \
    vim \
    && rm -rf /var/lib/apt/lists/*

# 创建虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安装依赖
COPY requirements.txt .
RUN pip install -r requirements.txt

# 复制源代码
COPY . /app
WORKDIR /app

# 开发模式启动
CMD ["python", "-m", "flask", "run", "--host=0.0.0.0", "--reload"]
```

## 热重载与代码映射

### 实时重载配置
```yaml
# docker-compose.override.yml
version: '3.8'

services:
  web:
    volumes:
      - .:/app  # 代码实时映射
      - /app/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true  # 文件监听优化

  api:
    volumes:
      - .:/code
    command: python -m flask run --host=0.0.0.0 --reload
```

### 文件监听优化
```bash
# 提高文件监听性能
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Docker Desktop 文件共享优化
# 在Docker Desktop设置中排除不必要的目录
```

## 调试配置

### Node.js 调试
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Docker: Node.js",
      "type": "node",
      "request": "attach",
      "port": 9229,
      "address": "localhost",
      "localRoot": "${workspaceFolder}",
      "remoteRoot": "/app",
      "restart": true
    }
  ]
}
```

### Python 调试
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Docker: Python",
      "type": "python",
      "request": "attach",
      "port": 5678,
      "host": "localhost",
      "pathMappings": [
        {
          "localRoot": "${workspaceFolder}",
          "remoteRoot": "/app"
        }
      ]
    }
  ]
}
```

## 开发工具集成

### 数据库管理工具
```yaml
# 数据库管理服务
services:
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "5050:80"
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@example.com
      - PGADMIN_DEFAULT_PASSWORD=password
    volumes:
      - pgadmin_data:/var/lib/pgadmin

  redis-commander:
    image: rediscommander/redis-commander
    ports:
      - "8081:8081"
    environment:
      - REDIS_HOSTS=local:redis:6379
```

### 监控与日志工具
```yaml
services:
  # 应用监控
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf

  # 日志收集
  logspout:
    image: gliderlabs/logspout
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: syslog://logstash:514
```

## 环境变量管理

### 多环境配置
```bash
# .env.development
NODE_ENV=development
DATABASE_URL=postgresql://user:pass@db:5432/dev
REDIS_URL=redis://redis:6379
DEBUG=true
LOG_LEVEL=debug

# .env.test
NODE_ENV=test
DATABASE_URL=postgresql://user:pass@db:5432/test
REDIS_URL=redis://redis:6379
DEBUG=false
LOG_LEVEL=info
```

### Docker Compose 环境变量
```yaml
# docker-compose.yml
services:
  app:
    environment:
      - NODE_ENV=${NODE_ENV:-development}
      - DATABASE_URL=${DATABASE_URL}
      - DEBUG=${DEBUG:-false}
    env_file:
      - .env
      - .env.${NODE_ENV:-development}
```

## 开发工作流

### Makefile 自动化
```makefile
# Makefile
.PHONY: dev build test clean

# 开发环境
dev:
	docker-compose -f docker-compose.dev.yml up --build

# 构建测试
build:
	docker-compose -f docker-compose.dev.yml build

# 运行测试
test:
	docker-compose -f docker-compose.dev.yml run --rm web npm test

# 清理环境
clean:
	docker-compose -f docker-compose.dev.yml down -v
	docker system prune -f

# 数据库迁移
migrate:
	docker-compose -f docker-compose.dev.yml run --rm web npm run migrate

# 进入容器
shell:
	docker-compose -f docker-compose.dev.yml exec web sh
```

### 开发脚本
```bash
#!/bin/bash
# dev.sh - 开发环境启动脚本

# 检查依赖
command -v docker >/dev/null 2>&1 || { echo "Docker required"; exit 1; }
command -v docker-compose >/dev/null 2>&1 || { echo "Docker Compose required"; exit 1; }

# 设置环境
export NODE_ENV=development
export COMPOSE_PROJECT_NAME=myapp_dev

# 启动服务
docker-compose -f docker-compose.dev.yml up --build "$@"
```

## 团队协作配置

### 统一开发环境
```bash
# .devcontainer/devcontainer.json
{
  "name": "Node.js Development",
  "dockerComposeFile": "../docker-compose.dev.yml",
  "service": "web",
  "workspaceFolder": "/app",
  "settings": {
    "terminal.integrated.shell.linux": "/bin/bash"
  },
  "extensions": [
    "dbaeumer.vscode-eslint",
    "ms-vscode.vscode-typescript-next"
  ],
  "forwardPorts": [3000, 9229]
}
```

### 预提交钩子
```bash
#!/bin/bash
# pre-commit.sh
docker-compose -f docker-compose.dev.yml run --rm web npm run lint
docker-compose -f docker-compose.dev.yml run --rm web npm test
```

## 性能优化配置

### 开发环境性能
```yaml
# docker-compose.override.yml
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

  db:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
```

### 缓存优化
```bash
# Docker BuildKit 启用
export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1

# 构建缓存配置
docker build --cache-from=myapp:latest -t myapp:dev .
```

## 故障排查与调试

### 开发环境问题解决
```bash
# 查看日志
docker-compose logs -f web
docker-compose logs --tail=100 db

# 进入容器调试
docker-compose exec web sh
docker-compose run --rm web bash

# 检查服务状态
docker-compose ps
docker-compose top

# 重启服务
docker-compose restart web
docker-compose up --force-recreate web
```

### 网络诊断
```bash
# 检查网络连接
docker-compose exec web ping db
docker-compose exec web nslookup db

# 端口检查
docker-compose port web 3000
netstat -tuln | grep 3000
```

> 提示：开发环境配置应该尽量接近生产环境，但保留调试和快速迭代的能力。

!> 重要：敏感信息（如密码、API密钥）应该通过环境变量或密钥管理工具管理，不要硬编码在配置文件中。