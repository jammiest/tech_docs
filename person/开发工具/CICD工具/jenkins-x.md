# Jenkins X

## 1. 概述

Jenkins X 是一个基于 Jenkins 的云原生 CI/CD 平台，专为 Kubernetes 环境设计。它提供了开箱即用的自动化流水线、GitOps 工作流和预览环境，简化了云原生应用的开发、测试和部署流程。

## 2. 核心概念

### 2.1 架构组件
```mermaid
graph TB
    A[Jenkins X Platform] --> B[DevPod Workspaces]
    A --> C[Automated Pipelines]
    A --> D[Preview Environments]
    A --> E[GitOps Promotion]
    
    F[Kubernetes Cluster] --> G[Jenkins X Operator]
    F --> H[Tekton Pipelines]
    F --> I[Prow GitOps]
    
    A --> F
    
    style A fill:#e1f5fe
    style F fill:#f3e5f5
```

### 2.2 关键特性
- **云原生**: 专为 Kubernetes 设计，完全容器化
- **GitOps 驱动**: 使用 Git 作为唯一事实来源
- **自动化流水线**: 基于 Tekton 的自动化 CI/CD
- **预览环境**: 自动为 PR 创建临时环境
- **环境 promotion**: 通过 GitOps 在不同环境间提升应用

## 3. 安装与配置

### 3.1 使用 jx CLI 安装
```bash
#!/bin/bash
# install-jenkins-x.sh

# 安装 jx CLI
curl -L https://github.com/jenkins-x/jx/releases/download/v3.10.0/jx-linux-amd64.tar.gz | tar xzv
sudo mv jx /usr/local/bin/

# 验证安装
jx version

# 创建 Kubernetes 集群（可选）
jx create cluster gke \
    --project-id my-project \
    --cluster-name jx-cluster \
    --region us-central1 \
    --machine-type n1-standard-4 \
    --min-num-nodes 3 \
    --max-num-nodes 10 \
    --zone us-central1-a

# 安装 Jenkins X
jx boot \
    --provider gke \
    --project-id my-project \
    --cluster-name jx-cluster \
    --git-username my-git-user \
    --git-token my-git-token \
    --domain my-domain.com

# 或者使用 Terraform
jx create cluster terraform \
    --skip-installation \
    --verbose
```

### 3.2 配置文件
```yaml
# jx-requirements.yml
cluster:
  clusterName: jx-cluster
  environmentGitOwner: my-org
  project: my-project
  provider: gke
  zone: us-central1-a

environments:
- key: dev
- key: staging
- key: production

ingress:
  domain: my-domain.com
  externalDNS: true
  tls:
    email: admin@my-domain.com
    enabled: true
    production: true

storage:
  logs:
    enabled: true
    url: gs://jx-logs
  reports:
    enabled: true
    url: gs://jx-reports
  repository:
    enabled: true
    url: gs://jx-repository

versionStream:
  url: https://github.com/jenkins-x/jenkins-x-versions.git
  ref: v1.0.0
```

## 4. 项目设置

### 4.1 创建新项目
```bash
#!/bin/bash
# create-jx-project.sh

# 从模板创建新项目
jx create quickstart \
    --filter golang-http \
    --project-name my-app \
    --batch-mode

# 或者从现有仓库导入
jx import \
    --url https://github.com/my-org/my-app.git \
    --pack go

# 查看项目信息
jx get activities -f my-app
jx get applications

# 创建自定义流水线
cat << EOF > jenkins-x.yml
buildPack: go
pipelineConfig:
  pipelines:
    release:
      pipeline:
        agent:
          image: golang:1.18
        stages:
        - name: build
          steps:
          - name: build-binaries
            command: make build
EOF
```

### 4.2 项目配置
```yaml
# charts/my-app/values.yaml
jenkins-x:
  pipeline:
    env:
    - name: GO_VERSION
      value: "1.18"
    - name: DOCKER_REGISTRY
      value: "gcr.io/my-project"

replicaCount: 2

image:
  repository: gcr.io/my-project/my-app
  tag: latest
  pullPolicy: Always

service:
  type: ClusterIP
  port: 8080

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
  - my-app.my-domain.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

## 5. 流水线配置

### 5.1 自动化流水线
```yaml
# .lighthouse/jenkins-x/release.yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: my-app-release
spec:
  pipelineSpec:
    workspaces:
    - name: source
    - name: docker-config
    params:
    - name: version
      type: string
    tasks:
    - name: build-test
      taskRef:
        name: go-build-test
      workspaces:
      - name: source
        workspace: source
      params:
      - name: package
        value: ./...
    - name: build-image
      taskRef:
        name: kaniko
      runAfter:
      - build-test
      workspaces:
      - name: source
        workspace: source
      - name: docker-config
        workspace: docker-config
      params:
      - name: IMAGE
        value: gcr.io/my-project/my-app:$(params.version)
    - name: deploy-staging
      taskRef:
        name: helm-deploy
      runAfter:
      - build-image
      workspaces:
      - name: source
        workspace: source
      params:
      - name: environment
        value: staging
      - name: version
        value: $(params.version)
```

### 5.2 自定义任务
```yaml
# .lighthouse/tasks/go-build-test.yaml
apiVersion: tekton.dev/v1beta1
kind: ClusterTask
metadata:
  name: go-build-test
spec:
  workspaces:
  - name: source
  params:
  - name: package
    type: string
    description: Go package to build and test
  steps:
  - name: build
    image: golang:1.18
    workingDir: $(workspaces.source.path)
    script: |
      #!/bin/sh
      go build -o bin/app $(params.package)
  - name: test
    image: golang:1.18
    workingDir: $(workspaces.source.path)
    script: |
      #!/bin/sh
      go test -v $(params.package) -coverprofile=coverage.out
  - name: coverage
    image: alpine:3.14
    workingDir: $(workspaces.source.path)
    script: |
      #!/bin/sh
      go tool cover -func=coverage.out
```

## 6. 环境管理

### 6.1 环境配置
```yaml
# env/staging/values.yaml
jenkins-x:
  pipeline:
    env:
    - name: ENVIRONMENT
      value: staging
    - name: API_URL
      value: https://api.staging.my-domain.com

expose:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

database:
  enabled: true
  url: postgres://user:pass@postgres-staging:5432/myapp
```

### 6.2 GitOps 工作流
```bash
#!/bin/bash
# gitops-promotion.sh

# 查看当前环境
jx get env

# 将应用提升到 staging 环境
jx promote my-app --version 1.2.0 --env staging

# 自动创建 PR 到 production
jx promote my-app --version 1.2.0 --env production --no-wait

# 查看 promotion 状态
jx get activities -f my-app

# 手动批准 production promotion
jx promote my-app --version 1.2.0 --env production --batch-mode

# 回滚到上一个版本
jx promote my-app --version 1.1.0 --env production --batch-mode
```

## 7. 预览环境

### 7.1 自动预览环境
```yaml
# .lighthouse/preview.yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: preview-environment
spec:
  pipelineSpec:
    workspaces:
    - name: source
    params:
    - name: PR_NUMBER
      type: string
    tasks:
    - name: create-preview
      taskRef:
        name: create-preview-environment
      workspaces:
      - name: source
        workspace: source
      params:
      - name: PR_NUMBER
        value: $(params.PR_NUMBER)
    - name: deploy-preview
      taskRef:
        name: helm-deploy-preview
      runAfter:
      - create-preview
      workspaces:
      - name: source
        workspace: source
      params:
      - name: PR_NUMBER
        value: $(params.PR_NUMBER)
```

### 7.2 预览环境配置
```yaml
# charts/preview/values.yaml
preview:
  enabled: true
  domain: preview.my-domain.com
  tls:
    enabled: true
    issuer: letsencrypt-staging

ingress:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

database:
  enabled: true
  instance: preview-db
  username: preview_user
  password: preview_pass

redis:
  enabled: true
  host: preview-redis
  port: 6379
```

## 8. 监控与日志

### 8.1 监控配置
```yaml
# charts/monitoring/values.yaml
prometheus:
  enabled: true
  scrapeInterval: 30s
  retention: 15d

grafana:
  enabled: true
  adminPassword: admin123
  datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    access: proxy

alertmanager:
  enabled: true
  config:
    global:
      smtp_smarthost: 'smtp.gmail.com:587'
      smtp_from: 'alerts@my-domain.com'
      smtp_auth_username: 'alerts@my-domain.com'
      smtp_auth_password: 'password'
    route:
      group_by: ['alertname']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 1h
      receiver: 'email-alerts'
    receivers:
    - name: 'email-alerts'
      email_configs:
      - to: 'devops@my-domain.com'
        send_resolved: true
```

### 8.2 日志收集
```yaml
# charts/logging/values.yaml
loki:
  enabled: true
  config:
    auth_enabled: false
    ingester:
      chunk_idle_period: 1h
      chunk_target_size: 1048576
    schema_config:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: s3
      schema: v11
      index:
        prefix: index_
        period: 24h
    storage_config:
      boltdb_shipper:
        active_index_directory: /var/loki/index
        cache_location: /var/loki/cache
        shared_store: s3
      aws:
        s3: s3://us-east-1/loki-bucket
        region: us-east-1

promtail:
  enabled: true
  config:
    server:
      http_listen_port: 3101
    positions:
      filename: /var/log/positions.yaml
    clients:
    - url: http://loki:3100/loki/api/v1/push
    scrape_configs:
    - job_name: kubernetes-pods
      kubernetes_sdiscover:
        - role: pod
      pipeline_stages:
      - docker: {}
      - cri: {}
      - regex:
          expression: '^(?P<timestamp>\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\.\d+Z)'
      - timestamp:
          source: timestamp
          format: RFC3339Nano
```

## 9. 高级配置

### 9.1 自定义构建包
```yaml
# buildpacks/my-buildpack/pipeline.yaml
buildPack: my-buildpack
pipelineConfig:
  pipelines:
    release:
      pipeline:
        agent:
          image: my-custom-builder
        stages:
        - name: custom-build
          steps:
          - name: build
            command: make all
            args:
            - BUILD_TYPE=release
        - name: security-scan
          steps:
          - name: scan
            command: security-scanner
            args:
            - --output
            - report.json
        - name: deploy
          steps:
          - name: deploy
            command: helm upgrade
            args:
            - --install
            - --namespace
            - $(env.NAMESPACE)
            - my-app
            - ./charts/my-app

filePatterns:
- "**/*.go"
- "go.mod"
- "go.sum"
- "Makefile"
```

### 9.2 扩展配置
```yaml
# jx-extensions.yaml
extensions:
- name: jx-preview
  version: 0.0.10
  repository: https://github.com/jenkins-x-charts/preview
- name: jx-kubeless
  version: 0.0.20
  repository: https://github.com/jenkins-x-charts/kubeless
- name: jx-knative
  version: 0.1.5
  repository: https://github.com/jenkins-x-charts/knative

dependencies:
- name: cert-manager
  version: 1.5.3
  repository: https://charts.jetstack.io
- name: nginx-ingress
  version: 3.34.0
  repository: https://kubernetes.github.io/ingress-nginx
- name: prometheus
  version: 15.5.3
  repository: https://prometheus-community.github.io/helm-charts

values:
  global:
    domain: my-domain.com
    storage:
      type: gcs
      bucket: jx-storage
  jenkins:
    enabled: false
  tekton:
    enabled: true
```
