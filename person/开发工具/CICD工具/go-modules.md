# Go Modules CI/CD 集成指南

## 1. Go Modules 基础配置

### 1.1 初始化模块
```bash
#!/bin/bash
# init-go-module.sh

# 初始化新模块
go mod init github.com/yourusername/yourproject

# 添加依赖
go get github.com/gin-gonic/gin@v1.8.1
go get github.com/stretchr/testify@v1.8.0

# 整理依赖
go mod tidy

# 验证依赖
go mod verify

# 查看依赖图
go mod graph

# 生成vendor目录
go mod vendor
```

### 1.2 go.mod 示例
```go
module github.com/yourusername/yourproject

go 1.20

require (
    github.com/gin-gonic/gin v1.8.1
    github.com/stretchr/testify v1.8.0
)

require (
    github.com/davecgh/go-spew v1.1.1 // indirect
    github.com/gin-contrib/sse v0.1.0 // indirect
    github.com/go-playground/locales v0.14.0 // indirect
    github.com/go-playground/universal-translator v0.18.0 // indirect
    github.com/go-playground/validator/v10 v10.10.0 // indirect
    github.com/goccy/go-json v0.9.7 // indirect
    github.com/json-iterator/go v1.1.12 // indirect
    github.com/leodido/go-urn v1.2.1 // indirect
    github.com/mattn/go-isatty v0.0.14 // indirect
    github.com/modern-go/concurrent v0.0.0-20180228061459-e0a39a4cb421 // indirect
    github.com/modern-go/reflect2 v1.0.2 // indirect
    github.com/pelletier/go-toml/v2 v2.0.1 // indirect
    github.com/pmezard/go-difflib v1.0.0 // indirect
    github.com/ugorji/go/codec v1.2.7 // indirect
    golang.org/x/crypto v0.0.0-20210711020723-a769d52b0f97 // indirect
    golang.org/x/net v0.0.0-20210226172049-e18ecbb05110 // indirect
    golang.org/x/sys v0.0.0-20210806184541-e5e7981a1069 // indirect
    golang.org/x/text v0.3.6 // indirect
    gopkg.in/yaml.v2 v2.4.0 // indirect
    gopkg.in/yaml.v3 v3.0.1 // indirect
)
```

## 2. GitHub Actions 集成

### 2.1 基础 CI 工作流
```yaml
# .github/workflows/go-ci.yml
name: Go CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        go: [ '1.19', '1.20', '1.21' ]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Go
      uses: actions/setup-go@v3
      with:
        go-version: ${{ matrix.go }}
    
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: |
          ~/go/pkg/mod
          ./vendor
        key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
    
    - name: Install dependencies
      run: |
        go mod download
        go mod vendor
    
    - name: Run tests
      run: |
        go test -v -race -coverprofile=coverage.out ./...
        go tool cover -html=coverage.out -o coverage.html
    
    - name: Upload coverage
      uses: actions/upload-artifact@v3
      with:
        name: coverage-report
        path: coverage.html
    
    - name: Upload test results
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: test-results-${{ matrix.go }}
        path: |
          coverage.out
          test.log
```

### 2.2 高级构建工作流
```yaml
# .github/workflows/go-build.yml
name: Go Build

on:
  release:
    types: [ created ]
  workflow_dispatch:

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    env:
      CGO_ENABLED: 0
      GOOS: linux
      GOARCH: amd64
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Go
      uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/go/pkg/mod
        key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
    
    - name: Build binary
      run: |
        go build -ldflags="-w -s" -o bin/app ./cmd/main.go
        tar -czf app.tar.gz bin/app
    
    - name: Upload artifact
      uses: actions/upload-artifact@v3
      with:
        name: app-binary
        path: app.tar.gz
    
    - name: Docker build
      uses: docker/build-push-action@v4
      with:
        context: .
        push: ${{ github.event_name == 'release' }}
        tags: |
          ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}:${{ github.sha }}
          ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}:latest
        labels: |
          org.opencontainers.image.source=${{ github.repository }}
          org.opencontainers.image.revision=${{ github.sha }}
```

## 3. GitLab CI 集成

### 3.1 基础配置
```yaml
# .gitlab-ci.yml
image: golang:1.20

stages:
  - test
  - build
  - deploy

variables:
  GOPATH: $CI_PROJECT_DIR/.go
  GO111MODULE: "on"
  CGO_ENABLED: "0"

before_script:
  - mkdir -p $GOPATH
  - export PATH=$GOPATH/bin:$PATH
  - go version
  - go env

test:
  stage: test
  script:
    - go mod download
    - go test -v -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out
  artifacts:
    paths:
      - coverage.out
    expire_in: 1 week

build:
  stage: build
  script:
    - go build -ldflags="-w -s" -o bin/app ./cmd/main.go
    - tar -czf app.tar.gz bin/app
  artifacts:
    paths:
      - app.tar.gz
    expire_in: 1 week

deploy:
  stage: deploy
  only:
    - tags
  script:
    - echo "Deploying version ${CI_COMMIT_TAG}"
    - ./deploy.sh ${CI_COMMIT_TAG}
```

### 3.2 多架构构建
```yaml
# .gitlab-ci.yml
build-multiarch:
  stage: build
  image: docker:20.10
  services:
    - docker:20.10-dind
  variables:
    DOCKER_BUILDKIT: 1
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker buildx create --use
    - docker buildx build
      --platform linux/amd64,linux/arm64
      --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      --tag $CI_REGISTRY_IMAGE:latest
      --push
      .
  only:
    - main
```

## 4. 私有模块配置

### 4.1 GOPROXY 配置
```bash
#!/bin/bash
# setup-go-proxy.sh

# 设置私有仓库
go env -w GOPRIVATE=github.com/yourcompany/*

# 配置多个代理
go env -w GOPROXY=https://goproxy.io,direct

# 配置认证
git config --global url."https://${GITHUB_TOKEN}:x-oauth-basic@github.com".insteadOf "https://github.com"

# 或者使用 .netrc
cat > ~/.netrc << EOF
machine github.com
login yourusername
password ${GITHUB_TOKEN}
EOF
```

### 4.2 CI 中的私有模块
```yaml
# .github/workflows/private-modules.yml
name: Private Modules

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v3
    
    - name: Setup Go
      uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    
    - name: Configure private modules
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      run: |
        git config --global url."https://${GITHUB_TOKEN}:x-oauth-basic@github.com".insteadOf "https://github.com"
        go env -w GOPRIVATE=github.com/yourcompany/*
    
    - name: Build
      run: |
        go mod download
        go build ./...
```

## 5. 版本管理与发布

### 5.1 语义化版本管理
```bash
#!/bin/bash
# release.sh

# 获取最新标签
LATEST_TAG=$(git describe --tags --abbrev=0)

# 生成新版本
if [[ $LATEST_TAG =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
    MAJOR=$(echo $LATEST_TAG | cut -d. -f1 | tr -d 'v')
    MINOR=$(echo $LATEST_TAG | cut -d. -f2)
    PATCH=$(echo $LATEST_TAG | cut -d. -f3)
    
    # 根据参数增加版本号
    case $1 in
        major)
            NEW_MAJOR=$((MAJOR+1))
            NEW_VERSION="v${NEW_MAJOR}.0.0"
            ;;
        minor)
            NEW_MINOR=$((MINOR+1))
            NEW_VERSION="v${MAJOR}.${NEW_MINOR}.0"
            ;;
        patch)
            NEW_PATCH=$((PATCH+1))
            NEW_VERSION="v${MAJOR}.${MINOR}.${NEW_PATCH}"
            ;;
        *)
            echo "Usage: $0 [major|minor|patch]"
            exit 1
            ;;
    esac
else
    NEW_VERSION="v1.0.0"
fi

# 创建标签并推送
git tag -a $NEW_VERSION -m "Release $NEW_VERSION"
git push origin $NEW_VERSION

# 更新版本信息
echo "package version" > version.go
echo "" >> version.go
echo "const Version = \"$NEW_VERSION\"" >> version.go
```

### 5.2 GoReleaser 配置
```yaml
# .goreleaser.yml
before:
  hooks:
    - go mod download
    - go generate ./...

builds:
  - env:
      - CGO_ENABLED=0
    goos:
      - linux
      - darwin
      - windows
    goarch:
      - amd64
      - arm64
    flags:
      - -trimpath
    ldflags:
      - -s -w -X main.version={{.Version}} -X main.commit={{.Commit}}

archives:
  - format: tar.gz
    name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"

checksum:
  name_template: "checksums.txt"

snapshot:
  name_template: "{{ .Tag }}-next"

release:
  github:
    owner: yourusername
    name: yourproject
  prerelease: auto
```

## 6. 安全扫描

### 6.1 依赖安全扫描
```yaml
# .github/workflows/go-security.yml
name: Go Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'
  push:
    branches: [ main ]

jobs:
  gosec:
    name: Run Gosec
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    - name: Install Gosec
      run: go install github.com/securego/gosec/v2/cmd/gosec@latest
    - name: Run Gosec
      run: gosec -fmt=json -out=results.json ./...
    - name: Upload results
      uses: actions/upload-artifact@v3
      with:
        name: gosec-results
        path: results.json

  govulncheck:
    name: Run Govulncheck
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    - name: Run Govulncheck
      run: go install golang.org/x/vuln/cmd/govulncheck@latest && govulncheck ./...
```

### 6.2 依赖漏洞检查
```bash
#!/bin/bash
# check-vulnerabilities.sh

# 安装 govulncheck
go install golang.org/x/vuln/cmd/govulncheck@latest

# 检查项目漏洞
govulncheck ./...

# 检查依赖漏洞
go list -m all | govulncheck -mode=mod

# 生成漏洞报告
govulncheck -json ./... > vuln-report.json

# 与已知漏洞数据库对比
curl -s https://vuln.go.dev/ID/GO-2022-1234 | jq .
```

## 7. 性能优化

### 7.1 构建优化
```bash
#!/bin/bash
# build-optimized.sh

# 静态链接
CGO_ENABLED=0 go build -a -installsuffix cgo -o app .

# 最小化二进制
go build -ldflags="-w -s" -o app .

# 使用 upx 压缩
upx --best --lzma app

# 分离调试信息
go build -ldflags="-w -s" -o app .
objcopy --only-keep-debug app app.debug
strip --strip-debug --strip-unneeded app
```

### 7.2 CI 中的构建缓存
```yaml
# .github/workflows/go-build.yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - uses: actions/cache@v3
      with:
        path: |
          ~/go/pkg/mod
          ./vendor
        key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
    
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
        cache: true
        cache-dependency-path: go.sum
    
    - run: go build -o app .
```

## 8. 多模块管理

### 8.1 Workspace 配置
```bash
#!/bin/bash
# setup-workspace.sh

# 初始化工作空间
go work init

# 添加子模块
go work use ./module1
go work use ./module2

# 同步依赖
go work sync

# 工作空间示例
cat > go.work << EOF
go 1.20

use (
    ./module1
    ./module2
    ./module3
)
EOF
```

### 8.2 多模块 CI 配置
```yaml
# .github/workflows/multi-module.yml
name: Multi-Module CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        module: [ "module1", "module2", "module3" ]
    
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    
    - name: Test ${{ matrix.module }}
      working-directory: ${{ matrix.module }}
      run: |
        go mod download
        go test -v ./...
    
    - name: Build ${{ matrix.module }}
      working-directory: ${{ matrix.module }}
      run: go build -o ../bin/${{ matrix.module }} .
```

## 9. 文档生成与发布

### 9.1 文档生成
```bash
#!/bin/bash
# generate-docs.sh

# 安装 godoc
go install golang.org/x/tools/cmd/godoc@latest

# 生成文档
godoc -http=:6060 &
DOC_PID=$!
sleep 5
wget --mirror --no-parent --page-requisites --convert-links http://localhost:6060/pkg/
kill $DOC_PID

# 生成 API 文档
go install github.com/swaggo/swag/cmd/swag@latest
swag init -g cmd/main.go

# 生成 Markdown 文档
go install github.com/princjef/gomarkdoc/cmd/gomarkdoc@latest
gomarkdoc --output docs/README.md ./...
```

### 9.2 文档发布
```yaml
# .github/workflows/docs.yml
name: Docs

on:
  push:
    branches: [ main ]
    paths: [ '**/*.go', '!vendor/**' ]

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    
    - name: Generate docs
      run: |
        go install golang.org/x/tools/cmd/godoc@latest
        go install github.com/swaggo/swag/cmd/swag@latest
        swag init -g cmd/main.go
        godoc -http=:6060 &
        DOC_PID=$!
        sleep 5
        wget --mirror --no-parent --page-requisites --convert-links http://localhost:6060/pkg/
        kill $DOC_PID
    
    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./localhost:6060
        keep_files: false
```
