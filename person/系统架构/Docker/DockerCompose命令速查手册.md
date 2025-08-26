# Docker Compose 命令速查手册

## 基本命令
```bash
# 启动服务
docker-compose up                      # 启动所有服务（前台运行）
docker-compose up -d                   # 后台启动所有服务
docker-compose up service1 service2    # 启动指定服务

# 停止服务
docker-compose down                    # 停止并删除所有容器、网络
docker-compose down -v                 # 同时删除数据卷
docker-compose down --rmi all          # 同时删除镜像

# 服务管理
docker-compose start                   # 启动已存在的服务容器
docker-compose stop                    # 停止服务容器
docker-compose restart                 # 重启服务容器
docker-compose pause                   # 暂停服务容器
docker-compose unpause                 # 恢复暂停的服务容器
```

## 查看和信息
```bash
# 查看状态
docker-compose ps                      # 查看服务状态
docker-compose ps service1             # 查看指定服务状态
docker-compose top                     # 显示运行中服务的进程

# 查看日志
docker-compose logs                    # 查看所有服务日志
docker-compose logs -f                 # 实时查看日志
docker-compose logs service1           # 查看指定服务日志
docker-compose logs -f service1        # 实时查看指定服务日志
docker-compose logs --tail=100         # 查看最后100行日志

# 信息查询
docker-compose config                  # 验证和查看compose文件
docker-compose config --services       # 显示所有服务名称
docker-compose config --volumes        # 显示所有数据卷
docker-compose images                  # 显示使用的镜像
docker-compose port service1 80        # 显示服务的端口映射
```

## 构建和镜像
```bash
# 构建相关
docker-compose build                   # 构建或重新构建服务镜像
docker-compose build --no-cache        # 构建时不使用缓存
docker-compose build service1          # 构建指定服务

# 镜像管理
docker-compose pull                    # 拉取服务镜像
docker-compose push                    # 推送服务镜像
```

## 执行和管理
```bash
# 执行命令
docker-compose exec service1 bash      # 在服务容器中执行命令
docker-compose exec -u root service1 bash # 以root用户执行
docker-compose exec -T service1 command # 不分配TTY执行命令

# 服务操作
docker-compose run service1 command    # 运行一次性命令
docker-compose run --rm service1 bash  # 运行后自动删除容器
docker-compose scale service1=3        # 扩展服务实例数量（v2）
docker-compose up --scale service1=3   # 扩展服务实例数量（v3）
```

## 环境和管理
```bash
# 环境变量
docker-compose --env-file .env.prod up # 使用特定环境文件

# 项目管理
docker-compose -p myproject up         # 指定项目名称
docker-compose -f docker-compose.prod.yml up # 使用特定compose文件
docker-compose -f docker-compose.yml -f docker-compose.override.yml up # 多文件组合
```

## 调试和监控
```bash
# 调试命令
docker-compose events                  # 查看实时事件
docker-compose kill                    # 强制停止服务容器
docker-compose kill -s SIGTERM         # 发送特定信号

# 资源查看
docker-compose stats                   # 显示容器资源使用统计
docker-compose stats service1          # 显示指定服务资源统计
```

## 常用组合命令
```bash
# 开发环境常用
docker-compose up -d --build           # 重新构建并后台启动
docker-compose up --force-recreate     # 强制重新创建容器
docker-compose up --no-deps service1   # 只启动服务，不启动依赖

# 生产环境常用
docker-compose -f docker-compose.prod.yml up -d
docker-compose -f docker-compose.prod.yml down -v
```

## Docker Compose 文件示例
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db
    volumes:
      - .:/app
    networks:
      - app-network

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

## 常用场景命令
```bash
# 开发环境重启单个服务
docker-compose up -d --no-deps --build web

# 查看服务IP地址
docker-compose exec web hostname -i

# 复制文件到容器
docker-compose cp localfile.txt web:/app/

# 从容器复制文件
docker-compose cp web:/app/logs.txt ./

# 查看服务环境变量
docker-compose exec web env

# 重新创建单个服务
docker-compose up -d --force-recreate web
```

## 故障排查命令
```bash
# 检查服务健康状态
docker-compose ps --filter "status=restarting"

# 查看服务退出代码
docker-compose ps -a

# 检查服务配置
docker-compose config --no-interpolate

# 查看服务依赖关系
docker-compose config | grep depends_on
```

## 版本差异说明
```bash
# V1 和 V2 语法差异
docker-compose scale web=3            # V1/V2 扩展实例
docker-compose up --scale web=3       # V3+ 扩展实例

# 版本兼容性
docker-compose --version              # 查看版本
docker-compose version                # 查看版本详细信息
```