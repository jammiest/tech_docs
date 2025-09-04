# Drone CI

## 1. 概述

Drone CI 是一个基于容器技术的轻量级持续集成平台，使用 Go 语言开发。它采用简单的 YAML 配置，支持 Docker、Kubernetes 等多种运行时，提供高度可扩展的流水线功能。

## 2. 核心概念

### 2.1 架构组件
```mermaid
graph TB
    A[Drone Server] --> B[Web Interface]
    A --> C[Pipeline Scheduler]
    A --> D[Secret Management]
    
    E[Drone Runner] --> F[Docker Runtime]
    E --> G[Kubernetes Runtime]
    E --> H[SSH Runtime]
    
    A --> E
    
    style A fill:#e1f5fe
    style E fill:#f3e5f5
```

### 2.2 关键特性
- **容器原生**: 所有步骤在隔离的容器中运行
- **简单配置**: 基于 YAML 的声明式配置
- **多运行时**: 支持 Docker, Kubernetes, Exec, SSH
- **插件系统**: 丰富的插件生态系统
- **轻量高效**: 资源占用低，启动速度快

## 3. 安装与配置

### 3.1 Docker 安装
```bash
#!/bin/bash
# install-drone.sh

# 创建网络
docker network create drone

# 启动 Drone Server
docker run -d \
  --name=drone \
  --network=drone \
  -v /var/lib/drone:/data \
  -e DRONE_GITEA_SERVER=https://gitea.example.com \
  -e DRONE_GITEA_CLIENT_ID=your-client-id \
  -e DRONE_GITEA_CLIENT_SECRET=your-client-secret \
  -e DRONE_RPC_SECRET=your-rpc-secret \
  -e DRONE_SERVER_HOST=drone.example.com \
  -e DRONE_SERVER_PROTO=https \
  -p 80:80 \
  -p 443:443 \
  drone/drone:2

# 启动 Drone Runner
docker run -d \
  --name=drone-runner \
  --network=drone \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=https \
  -e DRONE_RPC_HOST=drone.example.com \
  -e DRONE_RPC_SECRET=your-rpc-secret \
  -e DRONE_RUNNER_CAPACITY=2 \
  -e DRONE_RUNNER_NAME=linux-runner \
  drone/drone-runner-docker:1
```

### 3.2 Kubernetes 安装
```yaml
# drone-server.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drone-server
  namespace: drone
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drone-server
  template:
    metadata:
      labels:
        app: drone-server
    spec:
      containers:
      - name: drone-server
        image: drone/drone:2
        ports:
        - containerPort: 80
        - containerPort: 443
        env:
        - name: DRONE_GITEA_SERVER
          value: "https://gitea.example.com"
        - name: DRONE_GITEA_CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: drone-secrets
              key: client-id
        - name: DRONE_GITEA_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: drone-secrets
              key: client-secret"
        - name: DRONE_RPC_SECRET
          valueFrom:
            secretKeyRef:
              name: drone-secrets
              key: rpc-secret
        volumeMounts:
        - name: drone-data
          mountPath: /data
      volumes:
      - name: drone-data
        persistentVolumeClaim:
          claimName: drone-pvc
---
# drone-runner.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drone-runner
  namespace: drone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: drone-runner
  template:
    metadata:
      labels:
        app: drone-runner
    spec:
      serviceAccountName: drone-runner
      containers:
      - name: drone-runner
        image: drone/drone-runner-kube:1
        env:
        - name: DRONE_RPC_HOST
          value: "drone-server.drone.svc.cluster.local"
        - name: DRONE_RPC_SECRET
          valueFrom:
            secretKeyRef:
              name: drone-secrets
              key: rpc-secret
        - name: DRONE_RUNNER_CAPACITY
          value: "2"
```

## 4. 流水线配置

### 4.1 基础流水线
```yaml
# .drone.yml
kind: pipeline
type: docker
name: default

steps:
- name: checkout
  image: alpine
  commands:
  - mkdir -p /drone/src
  - cd /drone/src
  - git clone https://github.com/$${DRONE_REPO} .

- name: test
  image: node:18-alpine
  commands:
  - npm install
  - npm test
  - npm run coverage

- name: build
  image: node:18-alpine
  commands:
  - npm run build
  when:
    branch: main

- name: notify
  image: plugins/slack
  settings:
    webhook: https://hooks.slack.com/services/your/webhook
    channel: deployments
    username: drone
  when:
    status: [success, failure]

trigger:
  branch:
  - main
  - develop
  event:
  - push
  - pull_request
```

### 4.2 多阶段流水线
```yaml
# .drone.yml
kind: pipeline
type: docker
name: multi-stage

steps:
- name: setup
  image: alpine
  commands:
  - apk add --no-cache git curl

- name: unit-test
  image: node:18-alpine
  commands:
  - npm install
  - npm run test:unit
  - npm run coverage

- name: integration-test
  image: node:18-alpine
  commands:
  - npm run test:integration
  depends_on:
  - unit-test

- name: build-production
  image: node:18-alpine
  commands:
  - npm run build:prod
  when:
    branch: main
  depends_on:
  - integration-test

- name: deploy-staging
  image: plugins/kubectl
  settings:
    server: https://kubernetes.example.com
    token:
      from_secret: k8s_token
    namespace: staging
    command: apply -f deployment.yaml
  when:
    branch: develop
  depends_on:
  - build-production

- name: deploy-production
  image: plugins/kubectl
  settings:
    server: https://kubernetes.example.com
    token:
      from_secret: k8s_token
    namespace: production
    command: apply -f deployment.yaml
  when:
    branch: main
  depends_on:
  - build-production
```

## 5. 高级功能

### 5.1 矩阵构建
```yaml
# .drone.yml
kind: pipeline
type: docker
name: matrix-build

matrix:
  node_version:
  - 16
  - 18
  - 20
  os:
  - alpine
  - bullseye

steps:
- name: test
  image: node:$${node_version}-$${os}
  commands:
  - echo "Testing Node.js $${node_version} on $${os}"
  - npm install
  - npm test

- name: build
  image: node:$${node_version}-$${os}
  commands:
  - npm run build
  depends_on:
  - test
```

### 5.2 服务容器
```yaml
# .drone.yml
kind: pipeline
type: docker
name: with-services

services:
- name: database
  image: postgres:14
  environment:
    POSTGRES_DB: test
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test

- name: redis
  image: redis:7-alpine

steps:
- name: test-with-services
  image: node:18-alpine
  commands:
  - npm install
  - npm run test:integration
  environment:
    DATABASE_URL: postgres://test:test@database:5432/test
    REDIS_URL: redis://redis:6379
```

## 6. 插件系统

### 6.1 使用官方插件
```yaml
# .drone.yml
kind: pipeline
type: docker
name: plugins

steps:
- name: docker-build
  image: plugins/docker
  settings:
    repo: example/app
    tags: latest,${DRONE_COMMIT_SHA:0:8}
    username:
      from_secret: docker_username
    password:
      from_secret: docker_password

- name: slack-notify
  image: plugins/slack
  settings:
    webhook:
      from_secret: slack_webhook
    channel: deployments
    template: |
      Build {{ build.status }} for {{ repo.name }}
      Branch: {{ build.branch }}
      Commit: {{ build.commit }}

- name: deploy-kubernetes
  image: plugins/kubectl
  settings:
    server: https://kubernetes.example.com
    token:
      from_secret: k8s_token
    namespace: production
    command: apply -f deployment.yaml
```

### 6.2 自定义插件
```yaml
# .drone.yml
kind: pipeline
type: docker
name: custom-plugin

steps:
- name: custom-task
  image: your-registry/custom-plugin:latest
  settings:
    api_key:
      from_secret: api_key
    environment: production
    timeout: 300
```

## 7. 安全配置

### 7.1 密钥管理
```yaml
# .drone.yml
kind: pipeline
type: docker
name: secure

steps:
- name: deploy
  image: plugins/kubectl
  settings:
    server: https://kubernetes.example.com
    token:
      from_secret: k8s_token
    namespace: production
    command: apply -f deployment.yaml
  environment:
    ENV: production
    API_KEY:
      from_secret: api_key

- name: notify
  image: plugins/slack
  settings:
    webhook:
      from_secret: slack_webhook
    channel: deployments
```

### 7.2 安全扫描
```yaml
# .drone.yml
kind: pipeline
type: docker
name: security

steps:
- name: sast-scan
  image: aquasec/trivy:latest
  commands:
  - trivy fs --security-checks vuln,config .

- name: dependency-scan
  image: node:18-alpine
  commands:
  - npm audit --audit-level=high
  - npx snyk test --severity-threshold=high

- name: container-scan
  image: aquasec/trivy:latest
  commands:
  - trivy image your-registry/app:latest
```

## 8. 监控和优化

### 8.1 性能监控
```yaml
# .drone.yml
kind: pipeline
type: docker
name: monitoring

steps:
- name: performance-test
  image: node:18-alpine
  commands:
  - npm run test:performance
  - npm run lighthouse

- name: resource-monitor
  image: alpine
  commands:
  - echo "Memory usage:"
  - free -m
  - echo "Disk usage:"
  - df -h

- name: upload-metrics
  image: curlimages/curl:latest
  commands:
  - curl -X POST -d @metrics.json https://monitoring.example.com/api/metrics
```

### 8.2 缓存优化
```yaml
# .drone.yml
kind: pipeline
type: docker
name: optimized

steps:
- name: restore-cache
  image: alpine
  commands:
  - if [ -f /cache/node_modules.tar ]; then
      tar -xf /cache/node_modules.tar -C .
    fi

- name: install
  image: node:18-alpine
  commands:
  - npm install

- name: save-cache
  image: alpine
  commands:
  - tar -cf /cache/node_modules.tar node_modules
  when:
    status: success
```

## 9. 故障排除

### 9.1 调试配置
```yaml
# .drone.yml
kind: pipeline
type: docker
name: debug

steps:
- name: debug-info
  image: alpine
  commands:
  - echo "Drone environment variables:"
  - env | grep DRONE_
  - echo "Current directory:"
  - pwd
  - echo "Disk space:"
  - df -h
  - echo "Memory usage:"
  - free -m

- name: network-check
  image: alpine
  commands:
  - ping -c 3 github.com
  - nslookup github.com
  - curl -I https://github.com

- name: log-collector
  image: alpine
  commands:
  - mkdir -p /drone/logs
  - find /drone -name "*.log" -exec cp {} /drone/logs \;
  - tar -czf /drone/debug-logs.tar.gz /drone/logs
```

### 9.2 重试策略
```yaml
# .drone.yml
kind: pipeline
type: docker
name: retry

steps:
- name: flaky-test
  image: node:18-alpine
  commands:
  - for i in {1..3}; do
      npm run test:flaky && exit 0
      echo "Attempt $i failed, retrying..."
      sleep 5
    done
    exit 1

- name: reliable-build
  image: node:18-alpine
  commands:
  - npm run build
  retries: 2
  when:
    status: failure
```
