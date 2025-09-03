# Docker 安全实践完全指南

## 安全架构概述

```
+----------------+     +----------------+     +----------------+
|   主机安全     | --> |   镜像安全     | --> |   运行时安全   |
| - 内核加固    |     | - 漏洞扫描    |     | - 资源限制    |
| - 用户隔离    |     | - 最小化镜像  |     | - 能力控制    |
| - 网络隔离    |     | - 签名验证    |     | - 安全策略    |
+----------------+     +----------------+     +----------------+
```

## 主机安全加固

### 系统级安全配置
```bash
# 更新系统内核
sudo apt-get update && sudo apt-get upgrade

# 安装安全工具
sudo apt-get install auditd fail2ban unattended-upgrades

# 配置防火墙
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 2375/tcp  # Docker API端口（谨慎开放）
```

### Docker 守护进程安全
```json
// /etc/docker/daemon.json 安全配置
{
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/etc/docker/ca.pem",
  "tlscert": "/etc/docker/server-cert.pem",
  "tlskey": "/etc/docker/server-key.pem",
  "userns-remap": "default",
  "icc": false,
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true
}
```

### 用户命名空间隔离
```bash
# 启用用户命名空间映射
sudo echo "default:1000:65536" > /etc/subuid
sudo echo "default:1000:65536" > /etc/subgid

# 验证用户映射
docker run --rm alpine cat /proc/self/uid_map
```

## 镜像安全实践

### 安全镜像构建
```dockerfile
# 多阶段构建减少攻击面
FROM golang:1.18 AS builder
WORKDIR /app
COPY . .
RUN go build -o app .

FROM gcr.io/distroless/base
COPY --from=builder /app/app /
CMD ["/app"]

# 使用非root用户
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser

# 签名镜像
FROM alpine
# 构建后执行: docker trust sign myimage:latest
```

### 镜像漏洞扫描
```bash
# 使用 Docker Scan
docker scan myimage:latest

# 使用 Trivy
docker run --rm aquasec/trivy image myimage:latest

# 使用 Grype
docker run --rm anchore/grype myimage:latest

# 集成到CI/CD
docker build -t myapp .
docker scan myapp --file Dockerfile --exclude-base
```

### 镜像来源验证
```bash
# 启用内容信任
export DOCKER_CONTENT_TRUST=1

# 签名镜像
docker trust sign myimage:latest

# 验证签名
docker trust inspect myimage:latest

# 使用签名策略
docker trust signer add --key signer-key.pem signer-name myimage
```

## 容器运行时安全

### 安全运行配置
```bash
# 最小权限原则运行
docker run -d \
  --user 1000:1000 \
  --read-only \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  nginx:alpine

# 资源限制防止DoS
docker run -d \
  --memory 512m \
  --memory-swap 512m \
  --cpus 1.0 \
  --pids-limit 100 \
  myapp:latest
```

### 能力控制
```bash
# 删除所有能力，按需添加
docker run -d \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  nginx:alpine

# 危险能力列表（应避免）
--cap-add SYS_ADMIN     # 系统管理
--cap-add SYS_PTRACE    # 进程调试  
--cap-add SYS_MODULE    # 加载内核模块
--cap-add NET_RAW       # 原始网络访问
```

### 文件系统安全
```bash
# 只读根文件系统
docker run -d \
  --read-only \
  --tmpfs /tmp:rw,noexec,nodev,nosuid \
  myapp:latest

# 精确挂载控制
docker run -d \
  -v /app/data:/data:ro \          # 只读数据
  -v /app/config:/config:ro \      # 只读配置
  -v /app/tmp:/tmp:rw \            # 可写临时目录
  myapp:latest
```

## 网络安全

### 网络隔离策略
```bash
# 创建隔离网络
docker network create --internal isolated-net

# 使用自定义网络
docker run -d \
  --network isolated-net \
  --network-alias app \
  myapp:latest

# 限制网络访问
docker run -d \
  --network none \                  # 无网络
  --network mynet \                # 自定义网络
  --ip 172.18.0.10 \               # 固定IP
  --dns 8.8.8.8 \                  # 指定DNS
  myapp:latest
```

### 网络安全策略
```yaml
# docker-compose 网络安全
version: '3.8'

services:
  web:
    image: nginx:alpine
    networks:
      - frontend

  api:
    image: myapp:latest
    networks:
      - backend

  db:
    image: postgres:13
    networks:
      - database

networks:
  frontend:
    internal: false
  backend:
    internal: true
  database:
    internal: true
```

## 安全监控与审计

### 实时安全监控
```bash
# 监控容器行为
docker events --filter 'type=container'

# 审计日志配置
sudo auditctl -w /var/lib/docker -k docker
sudo auditctl -w /etc/docker -k docker

# 使用 Falco 监控
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /etc/falco/falco.yaml:/etc/falco/falco.yaml \
  falcosecurity/falco
```

### 安全扫描与检查
```bash
# 检查容器安全状态
docker container ls --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# 安全基准检查
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  docker/docker-bench-security

# CIS 基准检测
docker run --rm --net host --pid host --cap-add audit_control \
  -v /etc:/etc \
  -v /usr/lib:/usr/lib \
  aquasec/kube-bench:latest
```

## 密钥管理

### 安全密钥注入
```bash
# 使用 Docker Secrets（Swarm模式）
echo "mysecretpassword" | docker secret create db_password -

# 使用环境变量（谨慎）
docker run -d \
  -e DB_PASSWORD=$(cat /secrets/db_password) \
  myapp:latest

# 使用挂载文件
docker run -d \
  -v /secrets/db_password:/run/secrets/db_password:ro \
  myapp:latest
```

### 外部密钥管理
```bash
# 使用 HashiCorp Vault
docker run -d \
  -e VAULT_ADDR=https://vault.example.com \
  -e VAULT_TOKEN=$(vault read -field=token auth/token/lookup-self) \
  myapp:latest

# 使用 AWS Secrets Manager
docker run -d \
  -e AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} \
  -e AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} \
  -e AWS_REGION=us-east-1 \
  myapp:latest
```

## 安全策略实施

### 安全策略文件
```yaml
# security-policy.yml
version: "1.0"
settings:
  defaultSeccompProfile: "runtime/default"
  defaultAppArmorProfile: "docker-default"
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true

containers:
- name: "*"
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    capabilities:
      drop: ["ALL"]
      add: ["NET_BIND_SERVICE"]
```

### 策略执行工具
```bash
# 使用 Open Policy Agent
docker run -d \
  -v ./policies:/policies \
  openpolicyagent/opa:latest \
  run --server --set=plugins.docker.authz.allow_unqualified=false

# 使用 Aqua Security
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/aqua-enforcer:latest
```

## 应急响应

### 安全事件处理
```bash
# 隔离受影响容器
docker update --restart=no compromised_container
docker network disconnect bridge compromised_container

# 取证分析
docker export compromised_container > forensic.tar
docker inspect compromised_container > inspection.json
docker logs compromised_container > logs.txt

# 漏洞修复流程
1. 停止受影响容器
2. 更新基础镜像
3. 重新构建镜像
4. 部署新容器
5. 验证修复效果
```

### 安全审计日志
```bash
# 启用详细审计
sudo auditctl -a always,exit -F arch=b64 -S bind -S connect -k docker_network

# 监控敏感操作
sudo auditctl -w /usr/bin/docker -p x -k docker_cmd

# 审计日志分析
sudo ausearch -k docker | aureport -f -i
```

## 合规性与最佳实践

### CIS Docker 基准
```bash
# 自动化合规检查
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/target \
  aquasec/trivy config /target

# 安全配置验证
docker run --rm \
  -v /etc/docker:/etc/docker:ro \
  instrumenta/konfig-lint /etc/docker
```

### 安全开发流程
```yaml
# CI/CD 安全流水线
steps:
  - name: 代码扫描
    uses: sonarsource/sonarcloud-github-action@v1

  - name: 镜像构建
    run: docker build -t myapp .

  - name: 漏洞扫描
    run: docker scan myapp --file Dockerfile

  - name: 签名镜像
    run: docker trust sign myapp:latest

  - name: 安全部署
    run: docker run --security-opt no-new-privileges myapp:latest
```

> 提示：安全是一个持续的过程，需要定期审查和更新安全策略。

!> 重要：生产环境必须实施多层次的安全防护，包括网络隔离、最小权限和持续监控。