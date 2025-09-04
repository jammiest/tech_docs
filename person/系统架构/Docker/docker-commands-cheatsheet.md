# Docker 命令速查手册

## 镜像管理
```bash
# 查看镜像
docker images                          # 列出本地镜像
docker image ls                        # 同上
docker images -a                       # 列出所有镜像（包括中间层）

# 拉取和推送镜像
docker pull ubuntu:20.04               # 拉取镜像
docker push username/repo:tag          # 推送镜像到仓库

# 构建镜像
docker build -t myapp:1.0 .            # 构建镜像（当前目录Dockerfile）
docker build -f Dockerfile.dev -t myapp:dev . # 指定Dockerfile

# 删除镜像
docker rmi ubuntu:20.04                # 删除镜像
docker image prune                     # 删除悬空镜像
docker image prune -a                  # 删除所有未使用镜像

# 镜像历史和信息
docker history myapp:1.0               # 查看镜像构建历史
docker inspect myapp:1.0               # 查看镜像详细信息
```

## 容器管理
```bash
# 启动容器
docker run ubuntu:20.04                # 运行容器
docker run -it ubuntu:20.04 /bin/bash  # 交互式运行
docker run -d -p 80:80 nginx           # 后台运行并端口映射
docker run --name mynginx nginx        # 指定容器名称
docker run -v /host/path:/container/path nginx # 挂载数据卷

# 查看容器
docker ps                              # 查看运行中容器
docker ps -a                           # 查看所有容器（包括停止的）
docker ps -q                           # 只显示容器ID

# 停止和删除容器
docker stop container_id               # 停止容器
docker start container_id              # 启动已停止容器
docker restart container_id            # 重启容器
docker rm container_id                 # 删除容器
docker rm -f container_id              # 强制删除运行中容器
docker container prune                 # 删除所有停止的容器

# 容器操作
docker exec -it container_id /bin/bash # 进入运行中容器
docker logs container_id               # 查看容器日志
docker logs -f container_id            # 实时查看日志
docker inspect container_id            # 查看容器详细信息
docker stats                           # 查看容器资源使用情况
docker top container_id                # 查看容器内进程
```

## 网络管理
```bash
# 网络操作
docker network ls                      # 列出网络
docker network create mynetwork        # 创建网络
docker network inspect mynetwork       # 查看网络详情
docker network rm mynetwork            # 删除网络

# 容器网络
docker run --network=mynetwork nginx   # 指定网络运行容器
docker network connect mynetwork container_id # 连接容器到网络
docker network disconnect mynetwork container_id # 断开网络连接
```

## 数据卷管理
```bash
# 数据卷操作
docker volume ls                       # 列出数据卷
docker volume create myvolume          # 创建数据卷
docker volume inspect myvolume         # 查看数据卷详情
docker volume rm myvolume              # 删除数据卷
docker volume prune                    # 删除未使用数据卷

# 使用数据卷
docker run -v myvolume:/app/data nginx # 使用命名数据卷
docker run -v /host/path:/app/data nginx # 使用绑定挂载
```

## Docker Compose
```bash
# Compose 基本命令
docker-compose up                      # 启动所有服务
docker-compose up -d                   # 后台启动服务
docker-compose down                    # 停止并删除所有服务
docker-compose ps                      # 查看服务状态
docker-compose logs                    # 查看服务日志
docker-compose logs -f service_name    # 实时查看指定服务日志

# Compose 管理
docker-compose build                   # 构建服务镜像
docker-compose start                   # 启动已存在服务
docker-compose stop                    # 停止服务
docker-compose restart                 # 重启服务
docker-compose exec service_name bash  # 进入服务容器
docker-compose pull                    # 拉取服务镜像
```

## 系统管理
```bash
# Docker 系统信息
docker info                            # 显示Docker系统信息
docker version                         # 显示Docker版本信息
docker system df                       # 查看磁盘使用情况

# 清理资源
docker system prune                    # 删除所有未使用资源
docker system prune -a                 # 删除所有未使用资源（包括镜像）
docker container prune                 # 删除所有停止的容器
docker image prune                     # 删除悬空镜像
docker volume prune                    # 删除未使用数据卷
docker network prune                   # 删除未使用网络
```

## 实用技巧和高级命令
```bash
# 容器和主机文件拷贝
docker cp container_id:/path/to/file /host/path # 从容器拷贝到主机
docker cp /host/path container_id:/path/to/file # 从主机拷贝到容器

# 端口管理
docker port container_id               # 查看容器端口映射

# 环境变量
docker run -e VAR=value nginx          # 设置环境变量
docker run --env-file .env nginx       # 从文件设置环境变量

# 资源限制
docker run -m 512m nginx               # 内存限制
docker run --cpus=1.5 nginx            # CPU限制
docker run --memory-swap=1g nginx      # 内存+交换分区限制

# 重启策略
docker run --restart=always nginx      # 总是重启
docker run --restart=on-failure nginx  # 失败时重启
docker run --restart=unless-stopped nginx # 除非手动停止，否则总是重启

# 健康检查
docker run --health-cmd="curl -f http://localhost || exit 1" nginx
```

## Dockerfile 常用指令
```dockerfile
FROM ubuntu:20.04                      # 基础镜像
LABEL maintainer="your@email.com"      # 元数据
WORKDIR /app                           # 工作目录
COPY . /app                            # 拷贝文件
RUN apt-get update && apt-get install -y package  # 执行命令
EXPOSE 80                              # 暴露端口
ENV NODE_ENV=production                # 环境变量
CMD ["npm", "start"]                   # 容器启动命令
ENTRYPOINT ["/app/start.sh"]           # 入口点
USER nobody                            # 指定用户
VOLUME /data                           # 数据卷
```

## 常用镜像操作示例
```bash
# 运行MySQL
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  -v mysql_data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

# 运行Redis
docker run -d --name redis \
  -v redis_data:/data \
  -p 6379:6379 \
  redis:alpine

# 运行Nginx
docker run -d --name nginx \
  -v ./html:/usr/share/nginx/html \
  -p 80:80 \
  nginx:alpine

# 运行Node.js应用
docker run -d --name nodeapp \
  -v ./app:/app \
  -p 3000:3000 \
  -w /app \
  node:16 npm start
```

## 监控和调试
```bash
# 监控命令
docker events                          # 实时监控Docker事件
docker stats container_id              # 监控单个容器资源使用
docker attach container_id             # 附加到运行中容器

# 调试技巧
docker run --rm -it --entrypoint sh image_name # 以sh进入镜像
docker diff container_id               # 查看容器文件系统变化
docker commit container_id new_image_name # 从容器创建新镜像
```