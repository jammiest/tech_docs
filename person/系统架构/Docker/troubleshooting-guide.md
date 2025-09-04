# Docker 常见问题排查完全指南

## 问题分类索引

| 问题类型 | 常见症状 | 紧急程度 |
|---------|---------|---------|
| **容器启动失败** | Exit code 125/126/127 | 高 |
| **网络连接问题** | Connection refused/timeout | 高 |
| **存储卷问题** | Permission denied/No such file | 中 |
| **资源不足** | OOMKilled/No space left | 高 |
| **镜像问题** | Pull failed/Image not found | 中 |
| **性能问题** | High CPU/Memory usage | 中 |

## 容器生命周期问题

### 启动失败排查
```bash
# 查看容器退出代码
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.ExitCode}}"

# 常见退出代码含义
# 125: Docker守护进程错误
# 126: 命令执行错误  
# 127: 命令未找到
# 137: OOMKilled (内存不足)
# 139: Segmentation fault

# 调试启动过程
docker run --rm -it alpine sh
docker run --rm -it --entrypoint sh nginx:alpine
```

### 重启循环排查
```bash
# 查看重启策略
docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' container_name

# 检查重启次数
docker inspect -f '{{.RestartCount}}' container_name

# 禁用重启进行调试
docker update --restart=no container_name
docker logs container_name --tail=100
```

## 网络连接问题

### 基础网络排查
```bash
# 检查容器网络配置
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name

# 测试容器网络连通性
docker exec container_name ping -c 4 8.8.8.8
docker exec container_name nslookup google.com

# 检查端口映射
docker port container_name
netstat -tuln | grep 6379
```

### DNS 问题排查
```bash
# 检查容器DNS配置
docker exec container_name cat /etc/resolv.conf

# 测试DNS解析
docker exec container_name nslookup google.com
docker exec container_name dig @8.8.8.8 google.com

# 自定义DNS配置
docker run -d --dns 8.8.8.8 --dns 1.1.1.1 nginx:alpine
```

### 网络模式问题
```bash
# 检查网络模式
docker inspect -f '{{.HostConfig.NetworkMode}}' container_name

# 不同网络模式的连通性测试
docker run --rm --network host alpine ping -c 4 google.com
docker run --rm --network bridge alpine ping -c 4 google.com
docker run --rm --network none alpine ip addr show
```

## 存储卷问题

### 权限问题排查
```bash
# 检查卷权限
docker exec container_name ls -la /data

# 修复权限问题
docker run --rm -v myvolume:/data alpine chown -R 1000:1000 /data

# 查看卷映射
docker inspect -f '{{.Mounts}}' container_name
```

### 数据持久化问题
```bash
# 检查卷使用情况
docker volume ls
docker volume inspect volume_name

# 卷数据恢复
docker run --rm -v backup_volume:/backup -v production_volume:/data \
  alpine cp -a /backup/. /data/

# 卷空间清理
docker run --rm -v myvolume:/data alpine sh -c 'rm -rf /data/*'
```

## 资源限制问题

### 内存问题排查
```bash
# 检查内存限制
docker inspect -f '{{.HostConfig.Memory}}' container_name

# 监控内存使用
docker stats --format "table {{.Name}}\t{{.MemUsage}}\t{{.MemPerc}}"

# 处理OOMKilled
docker logs container_name | grep -i "out of memory"
docker update --memory 2g container_name
```

### CPU 问题排查
```bash
# 检查CPU限制
docker inspect -f '{{.HostConfig.NanoCpus}}' container_name

# 监控CPU使用
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.PIDs}}"

# 调整CPU限制
docker update --cpus 2.0 container_name
docker update --cpuset-cpus "0-3" container_name
```

### 磁盘空间问题
```bash
# 检查磁盘使用
docker system df
docker system df -v

# 清理无用资源
docker system prune -a
docker volume prune

# 检查大文件
find /var/lib/docker -type f -size +100M -exec ls -lh {} \;
```

## 镜像相关问题

### 镜像拉取失败
```bash
# 检查镜像标签
docker manifest inspect nginx:alpine

# 使用不同镜像源
docker pull registry.cn-hangzhou.aliyuncs.com/library/nginx:alpine

# 清理镜像缓存
docker image prune -a
```

### 镜像构建问题
```bash
# 详细构建输出
docker build --progress=plain -t myapp .

# 构建缓存调试
docker build --no-cache -t myapp .

# 多阶段构建调试
docker build --target builder -t myapp-builder .
```

## 性能问题排查

### 容器性能分析
```bash
# 实时性能监控
docker stats --all --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemPerc}}\t{{.NetIO}}\t{{.BlockIO}}"

# 进程级别监控
docker top container_name
docker exec container_name ps aux

# 系统调用跟踪
docker run --rm --cap-add=SYS_PTRACE alpine strace -p 1
```

### I/O 性能问题
```bash
# 磁盘I/O监控
docker exec container_name iostat -x 1
docker exec container_name iotop -o

# 网络I/O监控
docker exec container_name iftop
docker exec container_name nethogs
```

## 日志与调试

### 日志分析技巧
```bash
# 多容器日志跟踪
docker logs -f $(docker ps -q)

# 时间范围日志查询
docker logs --since "2023-01-01" --until "2023-01-02" container_name

# 日志模式匹配
docker logs container_name | grep -E "(ERROR|Exception|Failed)"
docker logs container_name | awk '{print $1}' | sort | uniq -c | sort -nr
```

### 高级调试方法
```bash
# 进入容器调试
docker exec -it container_name /bin/bash
docker run -it --rm --network container:container_name alpine sh

# 使用调试工具
docker run -it --rm --cap-add=SYS_PTRACE debug-tools:latest

# 核心转储分析
docker run --rm -v /tmp:/tmp alpine gdb /app/core
```

## Docker 守护进程问题

### 服务状态排查
```bash
# 检查Docker服务状态
sudo systemctl status docker
sudo journalctl -u docker.service -f

# 查看Docker信息
docker info
docker version

# 重启Docker服务
sudo systemctl restart docker
```

### API 连接问题
```bash
# 检查Docker socket权限
ls -la /var/run/docker.sock

# 测试API连接
curl --unix-socket /var/run/docker.sock http://v1.40/version

# 修复权限问题
sudo chmod 666 /var/run/docker.sock
```

## 常见错误解决方案

### 权限错误处理
```bash
# Permission denied 错误
docker run -u root container_name  # 临时解决方案
chmod 777 /host/path              # 修复主机权限

# SELinux 问题
docker run -v /host/path:/container/path:z container_name
```

### 端口冲突处理
```bash
# 端口已被占用
netstat -tuln | grep :80
sudo lsof -i :80

# 使用不同端口
docker run -p 8080:80 nginx:alpine

# 停止冲突容器
docker stop $(docker ps -q --filter "publish=80")
```

### 资源冲突处理
```bash
# 容器名称冲突
docker ps -a --filter "name=existing_name"
docker rm existing_name

# 卷名称冲突
docker volume ls --filter "name=existing_volume"
docker volume rm existing_volume
```

## 自动化排查脚本

### 健康检查脚本
```bash
#!/bin/bash
# 容器健康检查脚本
echo "=== Docker Container Health Check ==="

# 检查运行状态
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# 检查资源使用
echo -e "\n=== Resource Usage ==="
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemPerc}}"

# 检查健康状态
echo -e "\n=== Health Status ==="
docker ps --format "{{.Names}}" | while read container; do
    health=$(docker inspect --format='{{.State.Health.Status}}' $container 2>/dev/null || echo "N/A")
    echo "$container: $health"
done
```

### 故障诊断脚本
```bash
#!/bin/bash
# 自动故障诊断脚本
CONTAINER=$1

echo "Diagnosing container: $CONTAINER"

# 收集基本信息
echo "=== Basic Info ==="
docker inspect $CONTAINER | jq '.[] | {State, Config, NetworkSettings}'

# 检查日志
echo -e "\n=== Logs (last 50 lines) ==="
docker logs --tail=50 $CONTAINER

# 检查资源
echo -e "\n=== Resource Usage ==="
docker stats --no-stream $CONTAINER

# 检查进程
echo -e "\n=== Processes ==="
docker top $CONTAINER
```

## 预防性维护

### 定期检查任务
```bash
# 每日健康检查
docker system prune -f
docker image prune -a
docker volume prune

# 每周安全检查
docker scan $(docker images -q)
docker system df -v

# 每月性能优化
docker container ls --format "table {{.Names}}\t{{.RunningFor}}\t{{.Status}}"
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

> 提示：建立系统化的排查流程可以快速定位和解决Docker问题。

!> 重要：生产环境的问题排查应该遵循变更管理流程，避免误操作导致服务中断。