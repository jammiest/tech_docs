# Docker 生态工具完全指南

## 开发与调试工具

### Docker 扩展工具集
```bash
# 1. Docker Buildx (多架构构建)
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:multi-arch .

# 2. Docker Compose (多容器管理)
docker-compose up -d
docker-compose logs -f

# 3. Docker Scout (安全扫描)
docker scout quickview myapp:latest
docker scout cves myapp:latest

# 4. Docker Context (多环境管理)
docker context create production --docker "host=ssh://user@production"
docker context use production
```

### 开发效率工具
```bash
# 1. Dive - 镜像分析工具
docker run --rm -it wagoodman/dive:latest myapp:latest

# 2. ctop - 容器监控
docker run --rm -ti quay.io/vektorlab/ctop:latest

# 3. lazydocker - 终端UI管理
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock lazyteam/lazydocker

# 4. docui - 交互式管理
docker run --rm -it skanehira/docui
```

## 监控与日志工具

### 监控解决方案
```bash
# 1. cAdvisor - 容器监控
docker run -d \
  --name=cadvisor \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -v /sys:/sys:ro \
  -p 8080:8080 \
  gcr.io/cadvisor/cadvisor:latest

# 2. Prometheus + Grafana
docker run -d -p 9090:9090 prom/prometheus
docker run -d -p 3000:3000 grafana/grafana

# 3. NetData - 实时监控
docker run -d \
  --name=netdata \
  -p 19999:19999 \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  netdata/netdata
```

### 日志管理工具
```bash
# 1. ELK Stack
docker-compose -f elk-stack.yml up -d

# 2. Loki + Grafana
docker run -d -p 3100:3100 grafana/loki:latest
docker run -d -p 3000:3000 grafana/grafana

# 3. Fluentd
docker run -d \
  -v /var/log:/var/log \
  -p 24224:24224 \
  fluent/fluentd:latest
```

## 安全与扫描工具

### 安全扫描工具
```bash
# 1. Trivy - 漏洞扫描
docker run --rm aquasec/trivy image myapp:latest
docker run --rm aquasec/trivy fs --security-checks vuln,config .

# 2. Grype - 镜像扫描
docker run --rm anchore/grype myapp:latest

# 3. Clair - 静态分析
docker run -d -p 6060:6060 -p 6061:6061 quay.io/clair:v4.0

# 4. Docker Bench Security - 安全基准
docker run --rm --net host --pid host --cap-add audit_control \
  -v /etc:/etc \
  -v /usr/lib:/usr/lib \
  docker/docker-bench-security
```

### 运行时安全
```bash
# 1. Falco - 运行时安全监控
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /etc/falco/falco.yaml:/etc/falco/falco.yaml \
  falcosecurity/falco

# 2. Sysdig - 安全监控
docker run -d \
  --name sysdig \
  --privileged \
  -v /var/run/docker.sock:/host/var/run/docker.sock \
  -v /dev:/host/dev \
  sysdig/sysdig

# 3. Open Policy Agent - 策略执行
docker run -d -p 8181:8181 openpolicyagent/opa:latest
```

## 网络与存储工具

### 网络管理工具
```bash
# 1. Traefik - 反向代理
docker run -d \
  -p 80:80 \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  traefik:latest

# 2. Nginx Proxy Manager
docker run -d \
  -p 80:80 \
  -p 81:81 \
  -p 443:443 \
  jc21/nginx-proxy-manager:latest

# 3. Caddy - 现代Web服务器
docker run -d \
  -p 80:80 \
  -p 443:443 \
  caddy:latest
```

### 存储管理工具
```bash
# 1. Portworx - 容器存储
docker run -d \
  --name portworx \
  --privileged \
  -v /etc/pwx:/etc/pwx \
  portworx/px-base:latest

# 2. Rancher Longhorn - 分布式块存储
docker run -d \
  --name longhorn \
  longhornio/longhorn-manager:latest

# 3. MinIO - S3兼容对象存储
docker run -d \
  -p 9000:9000 \
  -p 9001:9001 \
  minio/minio:latest server /data --console-address ":9001"
```

## 编排与管理平台

### 容器编排工具
```bash
# 1. Kubernetes (minikube)
minikube start --driver=docker
minikube dashboard

# 2. Docker Swarm
docker swarm init
docker stack deploy -c docker-compose.yml myapp

# 3. Nomad
docker run -d \
  --name nomad \
  -p 4646:4646 \
  hashicorp/nomad:latest
```

### 管理平台
```bash
# 1. Portainer - 容器管理UI
docker run -d \
  -p 8000:8000 \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  portainer/portainer-ce:latest

# 2. Rancher - 多集群管理
docker run -d \
  -p 80:80 \
  -p 443:443 \
  rancher/rancher:latest

# 3. Lens IDE - Kubernetes IDE
# 桌面应用，提供Kubernetes集群管理
```

## 开发与测试工具

### 本地开发工具
```bash
# 1. Telepresence - 本地开发K8s
telepresence connect

# 2. Skaffold - 本地K8s开发
skaffold dev

# 3. Tilt - 微服务开发
tilt up

# 4. DevSpace - 云原生开发
devspace dev
```

### 测试工具
```bash
# 1. Testcontainers - 集成测试
docker run --rm testcontainers/example:latest

# 2. MailHog - 邮件测试
docker run -d -p 8025:8025 -p 1025:1025 mailhog/mailhog:latest

# 3. LocalStack - AWS模拟
docker run -d -p 4566:4566 localstack/localstack:latest

# 4. MockServer - API模拟
docker run -d -p 1080:1080 mockserver/mockserver:latest
```

## CI/CD 工具集成

### CI/CD 平台
```bash
# 1. Jenkins
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# 2. GitLab Runner
docker run -d \
  --name gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest

# 3. Drone CI
docker run -d \
  -p 80:80 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  drone/drone:latest
```

### 构建工具
```bash
# 1. BuildKit - 高级构建
DOCKER_BUILDKIT=1 docker build .

# 2. Kaniko - 无守护进程构建
docker run -v $(pwd):/workspace gcr.io/kaniko-project/executor:latest

# 3. Buildah - OCI镜像构建
buildah bud -t myapp:latest .

# 4. Img - 并行构建
img build -t myapp:latest .
```

## 数据库与中间件

### 开发数据库
```bash
# 1. PostgreSQL
docker run -d \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=password \
  postgres:13

# 2. MySQL
docker run -d \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0

# 3. Redis
docker run -d \
  -p 6379:6379 \
  redis:6-alpine

# 4. MongoDB
docker run -d \
  -p 27017:27017 \
  mongo:5
```

### 消息队列
```bash
# 1. RabbitMQ
docker run -d \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management

# 2. Kafka
docker run -d \
  -p 9092:9092 \
  apache/kafka:latest

# 3. Redis Streams
docker run -d \
  -p 6379:6379 \
  redis/redis-stack:latest
```

## 实用工具集合

### 运维工具
```bash
# 1. Heimdall - 应用仪表板
docker run -d \
  -p 80:80 \
  -p 443:443 \
  linuxserver/heimdall:latest

# 2. Watchtower - 自动更新
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower:latest

# 3. Docker-Slim - 镜像瘦身
docker-slim build myapp:latest

# 4. Trivy - 安全扫描
trivy image myapp:latest
```

### 诊断工具
```bash
# 1. netshoot - 网络诊断
docker run -it --rm nicolaka/netshoot

# 2. docker-debug - 调试容器
docker run -it --rm --cap-add SYS_PTRACE \
  -v /var/run/docker.sock:/var/run/docker.sock \
  goodwithtech/docker-debug:latest

# 3. ctop - 容器监控
docker run --rm -ti quay.io/vektorlab/ctop:latest
```

## 可视化与GUI工具

### Web管理界面
```bash
# 1. Portainer (社区版)
docker run -d \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  portainer/portainer-ce:latest

# 2. Docker UI
docker run -d \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  abh1nav/dockerui:latest

# 3. Yacht
docker run -d \
  -p 8000:8000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  selfhostedpro/yacht:latest
```

### 桌面应用
```bash
# 1. Docker Desktop (Mac/Windows)
# 官方桌面应用，包含完整Docker环境

# 2. Kitematic (已弃用，由Docker Desktop替代)
# 曾经的GUI管理工具

# 3. Lens IDE
# Kubernetes集群管理IDE
```

## 最佳实践工具链

### 推荐工具组合
```yaml
# 开发环境工具链
development:
  - Docker Desktop (本地开发)
  - Docker Compose (服务编排)
  - Skaffold (K8s开发)
  - Telepresence (远程调试)

# CI/CD 工具链
ci_cd:
  - GitHub Actions (CI平台)
  - Trivy (安全扫描)
  - Buildx (多架构构建)
  - Harbor (镜像仓库)

# 生产环境工具链
production:
  - Kubernetes (编排平台)
  - Prometheus + Grafana (监控)
  - ELK Stack (日志管理)
  - Falco (运行时安全)
```

### 工具选择建议
1. **小型项目**: Docker Compose + Portainer
2. **中型项目**: Docker Swarm + Traefik + ELK
3. **大型项目**: Kubernetes + Istio + Prometheus
4. **企业级**: Rancher + Harbor + GitLab CI

> 提示：根据项目规模和技术栈选择合适的工具组合，避免过度工程化。

!> 重要：生产环境工具选择应考虑社区支持、安全性和可维护性。