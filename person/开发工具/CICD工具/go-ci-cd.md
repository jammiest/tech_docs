# Go 项目 CI/CD

## 1. 概述

Go 语言的 CI/CD 流程因其编译型语言特性和简单的依赖管理而具有独特优势。本指南提供完整的 Go 项目 CI/CD 实施方案，涵盖代码检查、测试、构建、容器化和部署全流程。

## 2. 环境配置

### 2.1 基础环境设置
```bash
#!/bin/bash
# setup-go-environment.sh

# 安装最新版 Go
curl -OL https://golang.org/dl/go1.20.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
rm go1.20.linux-amd64.tar.gz

# 配置环境变量
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
source ~/.bashrc

# 安装常用工具
go install golang.org/x/tools/cmd/goimports@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install github.com/securego/gosec/v2/cmd/gosec@latest
go install github.com/go-critic/go-critic/cmd/gocritic@latest
go install gotest.tools/gotestsum@latest

# 验证安装
go version
golangci-lint --version
gosec --version
```

### 2.2 Docker 化 Go 环境
```dockerfile
# Dockerfile
# 构建阶段
FROM golang:1.20-alpine AS builder

WORKDIR /app

# 预装依赖工具
RUN apk add --no-cache git make curl

# 复制依赖文件
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码
COPY . .

# 构建应用
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /app/main ./cmd/main.go

# 最终阶段
FROM alpine:3.17

WORKDIR /app

# 从构建阶段复制二进制文件
COPY --from=builder /app/main /app/main
COPY --from=builder /app/configs /app/configs
COPY --from=builder /app/migrations /app/migrations

# 暴露端口
EXPOSE 8080

# 启动命令
CMD ["/app/main"]
```

## 3. CI/CD 流水线设计

### 3.1 GitHub Actions 配置
```yaml
# .github/workflows/go-ci-cd.yml
name: Go CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  GO_VERSION: '1.20'
  BINARY_NAME: 'myapp'
  DOCKER_IMAGE: 'ghcr.io/${{ github.repository_owner }}/${{ env.BINARY_NAME }}'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: ${{ env.GO_VERSION }}
      
      - name: Run golangci-lint
        run: golangci-lint run --timeout 5m
      
      - name: Run gosec
        run: gosec ./...
      
      - name: Run go-critic
        run: gocritic check ./...

  test:
    runs-on: ubuntu-latest
    needs: lint
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: --health-cmd="pg_isready" --health-interval=10s --health-timeout=5s --health-retries=3
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: ${{ env.GO_VERSION }}
      
      - name: Run tests
        run: gotestsum --format standard-verbose -- -coverprofile=coverage.out -race ./...
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: coverage.out

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: ${{ env.GO_VERSION }}
      
      - name: Build binary
        run: |
          CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o ${{ env.BINARY_NAME }} ./cmd/main.go
          mkdir -p dist
          mv ${{ env.BINARY_NAME }} dist/
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: ${{ env.BINARY_NAME }}-binary
          path: dist/${{ env.BINARY_NAME }}

  docker:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: ${{ env.BINARY_NAME }}-binary
          path: dist
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.ref == 'refs/heads/main' }}
          tags: ${{ env.DOCKER_IMAGE }}:${{ github.sha }}
          labels: |
            org.opencontainers.image.source=${{ github.repository }}
            org.opencontainers.image.revision=${{ github.sha }}
            org.opencontainers.image.created=${{ github.event.head_commit.timestamp }}

  deploy:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Install kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'latest'
      
      - name: Configure Kubernetes
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBE_CONFIG }}" > ~/.kube/config
          kubectl config use-context ${{ secrets.KUBE_CONTEXT }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/${{ env.BINARY_NAME }} ${{ env.BINARY_NAME }}=${{ env.DOCKER_IMAGE }}:${{ github.sha }} -n production
          kubectl rollout status deployment/${{ env.BINARY_NAME }} -n production
```

### 3.2 GitLab CI 配置
```yaml
# .gitlab-ci.yml
image: golang:1.20

stages:
  - lint
  - test
  - build
  - deploy

variables:
  GO_VERSION: "1.20"
  BINARY_NAME: "myapp"
  DOCKER_IMAGE: "$CI_REGISTRY_IMAGE/$BINARY_NAME"

before_script:
  - go version
  - go install golang.org/x/tools/cmd/goimports@latest
  - go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

lint:
  stage: lint
  script:
    - golangci-lint run --timeout 5m
    - gosec ./...
    - gocritic check ./...

test:
  stage: test
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_PASSWORD: "postgres"
    POSTGRES_DB: "testdb"
    REDIS_PASSWORD: "redis"
  script:
    - gotestsum --format standard-verbose -- -coverprofile=coverage.out -race ./...
    - go tool cover -func=coverage.out
  artifacts:
    paths:
      - coverage.out

build:
  stage: build
  script:
    - CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o $BINARY_NAME ./cmd/main.go
    - mkdir -p dist
    - mv $BINARY_NAME dist/
  artifacts:
    paths:
      - dist/$BINARY_NAME

docker-build:
  stage: build
  image: docker:20.10
  services:
    - docker:20.10-dind
  variables:
    DOCKER_BUILDKIT: 1
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build --build-arg VERSION=$CI_COMMIT_SHA -t $DOCKER_IMAGE:$CI_COMMIT_SHA .
    - docker push $DOCKER_IMAGE:$CI_COMMIT_SHA
  only:
    - main

deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment: production
  only:
    - main
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - kubectl set image deployment/$BINARY_NAME $BINARY_NAME=$DOCKER_IMAGE:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/$BINARY_NAME -n production
```

## 4. 代码质量工具

### 4.1 golangci-lint 配置
```yaml
# .golangci.yml
run:
  timeout: 5m
  modules-download-mode: readonly
  skip-dirs:
    - vendor
    - testdata
    - docs
  skip-files:
    - ".*_test.go"

linters:
  disable-all: true
  enable:
    - bodyclose
    - deadcode
    - depguard
    - dogsled
    - errcheck
    - goconst
    - gocritic
    - gofmt
    - goimports
    - gosec
    - govet
    - ineffassign
    - staticcheck
    - typecheck
    - unused
    - varcheck

linters-settings:
  gocritic:
    enabled-tags:
      - performance
      - style
      - experimental
  govet:
    check-shadowing: true
  depguard:
    list-type: blacklist
    include-go-root: true
    packages:
      - github.com/sirupsen/logrus
      - github.com/globalsign/mgo
  gosec:
    excludes:
      - G104 # Errors unhandled
      - G107 # URL string in variable

issues:
  max-per-linter: 0
  max-same-issues: 0
  exclude-use-default: false
```

### 4.2 代码格式化配置
```bash
#!/bin/bash
# format-code.sh

# 格式化代码
goimports -w -local "github.com/myorg" .

# 检查未格式化的文件
unformatted=$(goimports -l -local "github.com/myorg" .)
if [ -n "$unformatted" ]; then
  echo "以下文件需要格式化:"
  echo "$unformatted"
  exit 1
fi
```

## 5. 测试策略

### 5.1 单元测试最佳实践
```go
// service_test.go
package service

import (
	"context"
	"testing"
	"time"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/mock"
	"github.com/stretchr/testify/require"
)

type MockRepository struct {
	mock.Mock
}

func (m *MockRepository) GetUser(ctx context.Context, id int) (*User, error) {
	args := m.Called(ctx, id)
	return args.Get(0).(*User), args.Error(1)
}

func TestUserService_GetUser(t *testing.T) {
	t.Parallel()

	tests := []struct {
		name        string
		userID      int
		mockSetup   func(*MockRepository)
		expected    *User
		expectedErr error
	}{
		{
			name:   "success",
			userID: 1,
			mockSetup: func(mr *MockRepository) {
				mr.On("GetUser", mock.Anything, 1).
					Return(&User{ID: 1, Name: "John"}, nil)
			},
			expected: &User{ID: 1, Name: "John"},
		},
		{
			name:   "not found",
			userID: 2,
			mockSetup: func(mr *MockRepository) {
				mr.On("GetUser", mock.Anything, 2).
					Return(nil, ErrNotFound)
			},
			expectedErr: ErrNotFound,
		},
	}

	for _, tt := range tests {
		tt := tt
		t.Run(tt.name, func(t *testing.T) {
			t.Parallel()

			mockRepo := new(MockRepository)
			tt.mockSetup(mockRepo)

			svc := NewUserService(mockRepo)
			ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
			defer cancel()

			user, err := svc.GetUser(ctx, tt.userID)

			if tt.expectedErr != nil {
				require.ErrorIs(t, err, tt.expectedErr)
				return
			}

			require.NoError(t, err)
			assert.Equal(t, tt.expected, user)
			mockRepo.AssertExpectations(t)
		})
	}
}
```

### 5.2 集成测试配置
```go
// integration_test.go
package integration

import (
	"context"
	"database/sql"
	"os"
	"testing"
	"time"

	_ "github.com/lib/pq"
	"github.com/stretchr/testify/require"
	"github.com/testcontainers/testcontainers-go"
	"github.com/testcontainers/testcontainers-go/wait"
)

func TestUserRepository_Integration(t *testing.T) {
	if testing.Short() {
		t.Skip("跳过集成测试")
	}

	// 启动PostgreSQL容器
	ctx := context.Background()
	req := testcontainers.ContainerRequest{
		Image:        "postgres:15",
		ExposedPorts: []string{"5432/tcp"},
		Env: map[string]string{
			"POSTGRES_PASSWORD": "postgres",
			"POSTGRES_DB":       "testdb",
		},
		WaitingFor: wait.ForLog("database system is ready to accept connections").WithStartupTimeout(30 * time.Second),
	}
	pgContainer, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
		ContainerRequest: req,
		Started:         true,
	})
	require.NoError(t, err)
	defer pgContainer.Terminate(ctx)

	// 获取容器端口
	port, err := pgContainer.MappedPort(ctx, "5432")
	require.NoError(t, err)

	// 连接数据库
	dsn := "postgres://postgres:postgres@localhost:" + port.Port() + "/testdb?sslmode=disable"
	db, err := sql.Open("postgres", dsn)
	require.NoError(t, err)
	defer db.Close()

	// 执行迁移
	migrateSQL, err := os.ReadFile("testdata/migrations.sql")
	require.NoError(t, err)
	_, err = db.Exec(string(migrateSQL))
	require.NoError(t, err)

	// 测试代码...
}
```

## 6. 部署策略

### 6.1 多阶段部署
```bash
#!/bin/bash
# canary-deploy.sh

set -e

# 参数检查
if [ -z "$1" ]; then
  echo "用法: $0 <镜像版本>"
  exit 1
fi

IMAGE_VERSION=$1
DEPLOYMENT_NAME="myapp"
NAMESPACE="production"
CANARY_PERCENTAGE=10

echo "开始金丝雀部署 $IMAGE_VERSION..."

# 获取当前副本数
REPLICAS=$(kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.spec.replicas}')

# 计算金丝雀副本数
CANARY_REPLICAS=$(( (REPLICAS * CANARY_PERCENTAGE + 99) / 100 ))
if [ $CANARY_REPLICAS -lt 1 ]; then
  CANARY_REPLICAS=1
fi

# 创建金丝雀部署
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${DEPLOYMENT_NAME}-canary
  namespace: ${NAMESPACE}
  labels:
    app: ${DEPLOYMENT_NAME}
    track: canary
spec:
  replicas: ${CANARY_REPLICAS}
  selector:
    matchLabels:
      app: ${DEPLOYMENT_NAME}
      track: canary
  template:
    metadata:
      labels:
        app: ${DEPLOYMENT_NAME}
        track: canary
    spec:
      containers:
      - name: ${DEPLOYMENT_NAME}
        image: ${DOCKER_IMAGE}:${IMAGE_VERSION}
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
EOF

echo "金丝雀部署完成，监控指标..."

# 等待金丝雀就绪
kubectl rollout status deployment/${DEPLOYMENT_NAME}-canary -n $NAMESPACE --timeout=300s

# 验证金丝雀健康状态
CANARY_STATUS=$(kubectl get pods -n $NAMESPACE -l app=$DEPLOYMENT_NAME,track=canary -o jsonpath='{.items[*].status.conditions[?(@.type=="Ready")].status}')
if [ "$CANARY_STATUS" != "True" ]; then
  echo "金丝雀部署不健康，终止流程"
  exit 1
fi

echo "金丝雀验证通过，开始全量部署..."

# 更新主部署
kubectl set image deployment/$DEPLOYMENT_NAME $DEPLOYMENT_NAME=$DOCKER_IMAGE:$IMAGE_VERSION -n $NAMESPACE
kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE --timeout=600s

# 清理金丝雀部署
kubectl delete deployment ${DEPLOYMENT_NAME}-canary -n $NAMESPACE

echo "部署完成！"
```

### 6.2 回滚脚本
```bash
#!/bin/bash
# rollback-deployment.sh

set -e

DEPLOYMENT_NAME="myapp"
NAMESPACE="production"

echo "开始回滚部署..."

# 获取当前部署版本
CURRENT_IMAGE=$(kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].image}')

# 获取历史版本
HISTORY=$(kubectl rollout history deployment/$DEPLOYMENT_NAME -n $NAMESPACE --revision=0)
if [ -z "$HISTORY" ]; then
  echo "没有可用的历史版本"
  exit 1
fi

# 执行回滚
kubectl rollout undo deployment/$DEPLOYMENT_NAME -n $NAMESPACE
kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE --timeout=300s

NEW_IMAGE=$(kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].image}')

echo "回滚完成！"
echo "原版本: $CURRENT_IMAGE"
echo "新版本: $NEW_IMAGE"
```

## 7. 监控与日志

### 7.1 健康检查端点
```go
// health.go
package api

import (
	"net/http"
	"time"

	"github.com/labstack/echo/v4"
	"github.com/sirupsen/logrus"
)

type HealthCheckResponse struct {
	Status    string            `json:"status"`
	Timestamp time.Time         `json:"timestamp"`
	Services  map[string]string `json:"services"`
}

func (s *Server) healthCheckHandler(c echo.Context) error {
	ctx := c.Request().Context()
	response := HealthCheckResponse{
		Status:    "healthy",
		Timestamp: time.Now().UTC(),
		Services:  make(map[string]string),
	}

	// 检查数据库连接
	if err := s.db.PingContext(ctx); err != nil {
		response.Status = "unhealthy"
		response.Services["database"] = "unavailable"
		logrus.WithError(err).Error("数据库健康检查失败")
	} else {
		response.Services["database"] = "available"
	}

	// 检查Redis连接
	if _, err := s.redis.Ping(ctx).Result(); err != nil {
		response.Status = "unhealthy"
		response.Services["redis"] = "unavailable"
		logrus.WithError(err).Error("Redis健康检查失败")
	} else {
		response.Services["redis"] = "available"
	}

	if response.Status == "unhealthy" {
		return c.JSON(http.StatusServiceUnavailable, response)
	}

	return c.JSON(http.StatusOK, response)
}
```

### 7.2 性能监控配置
```yaml
# config/prometheus.yml
scrape_configs:
  - job_name: 'go_app'
    scrape_interval: 15s
    static_configs:
      - targets: ['localhost:8080']
    
    metrics_path: '/metrics'
    
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: prometheus-server:9090

  - job_name: 'go_runtime'
    scrape_interval: 15s
    static_configs:
      - targets: ['localhost:8081']
    
    metrics_path: '/debug/metrics'
    
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: prometheus-server:9090
```

## 8. 安全最佳实践

### 8.1 安全扫描集成
```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'  # 每周日运行
  push:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.20'
      
      - name: Run gosec
        run: |
          go install github.com/securego/gosec/v2/cmd/gosec@latest
          gosec -exclude=G104,G107 ./...
      
      - name: Run dependency-check
        uses: dependency-check/DependencyCheck@main
        with:
          project: 'Go Project'
          path: '.'
          format: 'HTML'
          out: 'reports'
      
      - name: Run trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload security report
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: reports
```

### 8.2 安全加固配置
```bash
#!/bin/bash
# secure-build.sh

set -e

# 启用Go的安全编译选项
export CGO_ENABLED=0
export GOOS=linux
export GOFLAGS="-buildvcs=false -trimpath"

# 构建时移除调试信息
LDFLAGS="-w -s -extldflags '-static'"

# 构建二进制文件
go build -ldflags="$LDFLAGS" -o bin/myapp ./cmd/main.go

# 检查二进制文件安全性
file bin/myapp | grep -q "statically linked"
if [ $? -ne 0 ]; then
  echo "错误：二进制文件不是静态链接"
  exit 1
fi

# 检查文件权限
find bin/ -type f -exec stat -c "%a %n" {} \; | grep -v "755"
if [ $? -eq 0 ]; then
  echo "错误：发现不安全的文件权限"
  exit 1
fi

echo "安全构建完成"
```
