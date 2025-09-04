# Go 测试与 CI/CD 集成指南

## 1. 基础测试配置

### 1.1 测试文件结构
```bash
# 标准测试文件命名
project/
├── main.go
├── main_test.go        # 单元测试
├── integration_test.go  # 集成测试
└── e2e/                # 端到端测试
    └── api_test.go
```

### 1.2 基础测试命令
```bash
#!/bin/bash
# basic-test-commands.sh

# 运行所有测试
go test ./...

# 显示详细输出
go test -v ./...

# 运行特定包的测试
go test ./pkg/utils

# 运行单个测试文件
go test ./pkg/utils -run ^TestParseDate$

# 运行匹配模式的测试
go test -run TestParse.* ./...

# 显示测试覆盖率
go test -cover ./...

# 生成覆盖率文件
go test -coverprofile=coverage.out ./...

# 查看覆盖率详情
go tool cover -func=coverage.out

# HTML格式覆盖率报告
go tool cover -html=coverage.out -o coverage.html

# 基准测试
go test -bench=. ./...
go test -bench=BenchmarkParseDate ./pkg/utils

# 基准测试+内存分析
go test -bench=. -benchmem ./...

# 测试超时控制
go test -timeout 30s ./...
```

## 2. 高级测试技术

### 2.1 表格驱动测试
```go
// pkg/calculator/calculator_test.go
package calculator

import "testing"

func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -1, -1, -2},
        {"mixed numbers", -1, 1, 0},
        {"zero values", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

### 2.2 测试辅助工具
```go
// testutils/testhelpers.go
package testutils

import (
    "io"
    "net/http"
    "net/http/httptest"
    "testing"
)

func NewTestServer(t *testing.T, handler http.Handler) *httptest.Server {
    t.Helper()
    return httptest.NewServer(handler)
}

func ReadTestData(t *testing.T, filename string) []byte {
    t.Helper()
    data, err := os.ReadFile(filepath.Join("testdata", filename))
    if err != nil {
        t.Fatalf("failed to read test data: %v", err)
    }
    return data
}

func AssertEqual(t *testing.T, got, want interface{}) {
    t.Helper()
    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v, want %v", got, want)
    }
}
```

## 3. CI/CD 集成配置

### 3.1 GitHub Actions 配置
```yaml
# .github/workflows/go-test.yml
name: Go Test

on: [push, pull_request]

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    strategy:
      matrix:
        go: ['1.19', '1.20', '1.21']
        os: [ubuntu-latest, macos-latest, windows-latest]
    
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
        go test -v -race -coverprofile=coverage-${{ matrix.go }}.out ./...
        go tool cover -func=coverage-${{ matrix.go }}.out
    
    - name: Upload coverage
      uses: actions/upload-artifact@v3
      with:
        name: coverage-report-${{ matrix.go }}-${{ matrix.os }}
        path: coverage-${{ matrix.go }}.out
    
    - name: Upload test results
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: test-results-${{ matrix.go }}-${{ matrix.os }}
        path: test.log
```

### 3.2 GitLab CI 配置
```yaml
# .gitlab-ci.yml
image: golang:1.20

stages:
  - test
  - build

variables:
  GOPATH: "$CI_PROJECT_DIR/.go"
  GO111MODULE: "on"
  CGO_ENABLED: "0"

before_script:
  - mkdir -p $GOPATH
  - export PATH=$GOPATH/bin:$PATH
  - go version
  - go env

test:
  stage: test
  services:
    - postgres:13
    - redis:7
  variables:
    POSTGRES_PASSWORD: "postgres"
    POSTGRES_DB: "testdb"
    REDIS_PASSWORD: "redis"
    TEST_DB_URL: "postgres://postgres:postgres@postgres:5432/testdb?sslmode=disable"
  script:
    - go test -v -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out
    - go tool cover -html=coverage.out -o coverage.html
  artifacts:
    paths:
      - coverage.out
      - coverage.html
    expire_in: 1 week

integration-test:
  stage: test
  script:
    - go test -tags=integration -v -race ./...
  only:
    - main
    - merge_requests
```

## 4. 高级测试策略

### 4.1 并行测试控制
```go
// pkg/service/concurrent_test.go
package service

import (
    "testing"
    "time"
)

func TestParallel(t *testing.T) {
    t.Parallel() // 标记为可并行运行
    
    tests := []struct {
        name string
        wait time.Duration
    }{
        {"test1", 1 * time.Second},
        {"test2", 2 * time.Second},
        {"test3", 3 * time.Second},
    }

    for _, tt := range tests {
        tt := tt // 创建局部变量副本
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel() // 子测试并行
            time.Sleep(tt.wait)
            // 测试逻辑...
        })
    }
}
```

### 4.2 测试环境管理
```go
// testutils/testenv.go
package testutils

import (
    "os"
    "testing"
)

type TestEnv struct {
    DBURL      string
    RedisURL   string
    TempDir    string
    CleanupFns []func()
}

func SetupTestEnv(t *testing.T) *TestEnv {
    t.Helper()
    
    env := &TestEnv{
        DBURL:    os.Getenv("TEST_DB_URL"),
        RedisURL: os.Getenv("TEST_REDIS_URL"),
    }
    
    // 创建临时目录
    tempDir, err := os.MkdirTemp("", "test-")
    if err != nil {
        t.Fatalf("failed to create temp dir: %v", err)
    }
    env.TempDir = tempDir
    env.AddCleanup(func() { os.RemoveAll(tempDir) })
    
    // 初始化测试数据库
    if env.DBURL == "" {
        env.DBURL = setupTestDB(t)
        env.AddCleanup(func() { cleanupTestDB(t, env.DBURL) })
    }
    
    return env
}

func (e *TestEnv) AddCleanup(fn func()) {
    e.CleanupFns = append(e.CleanupFns, fn)
}

func (e *TestEnv) Cleanup() {
    for i := len(e.CleanupFns) - 1; i >= 0; i-- {
        e.CleanupFns[i]()
    }
}
```

## 5. 性能测试与分析

### 5.1 基准测试示例
```go
// pkg/algorithm/sort_test.go
package algorithm

import (
    "math/rand"
    "testing"
    "time"
)

func generateRandomSlice(n int) []int {
    rand.Seed(time.Now().UnixNano())
    slice := make([]int, n)
    for i := 0; i < n; i++ {
        slice[i] = rand.Intn(1000)
    }
    return slice
}

func BenchmarkQuickSort100(b *testing.B) {
    for i := 0; i < b.N; i++ {
        b.StopTimer()
        slice := generateRandomSlice(100)
        b.StartTimer()
        QuickSort(slice)
    }
}

func BenchmarkQuickSort10000(b *testing.B) {
    for i := 0; i < b.N; i++ {
        b.StopTimer()
        slice := generateRandomSlice(10000)
        b.StartTimer()
        QuickSort(slice)
    }
}
```

### 5.2 性能分析集成
```yaml
# .github/workflows/benchmark.yml
name: Benchmark

on:
  schedule:
    - cron: '0 0 * * 0'  # 每周运行
  workflow_dispatch:

jobs:
  benchmark:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    
    - name: Run benchmarks
      run: |
        go test -bench=. -benchmem -cpuprofile=cpu.out -memprofile=mem.out ./...
        go tool pprof -top cpu.out
        go tool pprof -top mem.out
    
    - name: Upload profiles
      uses: actions/upload-artifact@v3
      with:
        name: performance-profiles
        path: |
          cpu.out
          mem.out
```

## 6. 测试覆盖率集成

### 6.1 Codecov 集成
```yaml
# .github/workflows/coverage.yml
name: Coverage

on: [push, pull_request]

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    
    - name: Run tests with coverage
      run: |
        go test -coverprofile=coverage.out -covermode=atomic ./...
    
    - name: Upload to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: coverage.out
        flags: unittests
        name: codecov-umbrella
```

### 6.2 覆盖率徽章生成
```bash
#!/bin/bash
# generate-coverage-badge.sh

# 安装 coverage 工具
go install github.com/AlekSi/gocoverutil@latest

# 生成覆盖率报告
go test -coverprofile=coverage.out ./...

# 生成覆盖率百分比
COVERAGE=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//')

# 生成徽章
curl -s "https://img.shields.io/badge/coverage-$COVERAGE%25-green" > coverage.svg

# 上传到 GitHub Pages
git add coverage.svg
git commit -m "Update coverage badge"
git push
```

## 7. 测试依赖管理

### 7.1 测试依赖隔离
```go
// internal/db/db_test.go
package db

import (
    "os"
    "testing"
    
    "github.com/DATA-DOG/go-sqlmock"
    "github.com/stretchr/testify/require"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func TestMain(m *testing.M) {
    // 全局测试初始化
    setupTestDB()
    
    // 运行测试
    code := m.Run()
    
    // 全局清理
    teardownTestDB()
    
    os.Exit(code)
}

func setupTestDB() {
    // 初始化测试数据库连接
}

func teardownTestDB() {
    // 清理测试数据库
}

func TestWithMock(t *testing.T) {
    // 创建 mock 数据库
    db, mock, err := sqlmock.New()
    require.NoError(t, err)
    defer db.Close()
    
    // 设置 mock 期望
    mock.ExpectQuery("SELECT \\* FROM users WHERE id = \\?").
        WithArgs(1).
        WillReturnRows(sqlmock.NewRows([]string{"id", "name"}).AddRow(1, "John"))
    
    // 使用 mock 数据库进行测试
    gormDB, err := gorm.Open(postgres.New(postgres.Config{
        Conn: db,
    }), &gorm.Config{})
    require.NoError(t, err)
    
    // 测试逻辑...
}
```

### 7.2 测试容器管理
```go
// testutils/testcontainers.go
package testutils

import (
    "context"
    "testing"
    "time"
    
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/wait"
)

type TestContainer struct {
    C testcontainers.Container
    URL string
}

func StartPostgresContainer(t *testing.T) *TestContainer {
    t.Helper()
    
    ctx := context.Background()
    req := testcontainers.ContainerRequest{
        Image:        "postgres:13",
        ExposedPorts: []string{"5432/tcp"},
        Env: map[string]string{
            "POSTGRES_PASSWORD": "postgres",
            "POSTGRES_USER":     "postgres",
            "POSTGRES_DB":       "testdb",
        },
        WaitingFor: wait.ForLog("database system is ready to accept connections").
            WithOccurrence(2).
            WithStartupTimeout(30 * time.Second),
    }
    
    pg, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: req,
        Started:         true,
    })
    if err != nil {
        t.Fatalf("failed to start postgres container: %v", err)
    }
    
    port, err := pg.MappedPort(ctx, "5432")
    if err != nil {
        t.Fatalf("failed to get postgres port: %v", err)
    }
    
    url := "postgres://postgres:postgres@localhost:" + port.Port() + "/testdb?sslmode=disable"
    
    return &TestContainer{
        C:   pg,
        URL: url,
    }
}
```

## 8. 测试报告与可视化

### 8.1 JUnit 报告生成
```yaml
# .github/workflows/junit-report.yml
name: JUnit Report

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-go@v3
      with:
        go-version: '1.20'
    
    - name: Install gotestsum
      run: go install gotest.tools/gotestsum@latest
    
    - name: Run tests with JUnit output
      run: |
        gotestsum --junitfile junit.xml --format standard-verbose
        gotestsum --junitfile junit.xml -- -coverprofile=coverage.out ./...
    
    - name: Upload JUnit report
      uses: actions/upload-artifact@v3
      with:
        name: test-report
        path: junit.xml
```

### 8.2 HTML 测试报告
```bash
#!/bin/bash
# generate-html-report.sh

# 安装测试报告工具
go install github.com/tebeka/go2xunit@latest
go install github.com/axw/gocov/gocov@latest
go install github.com/AlekSi/gocoverutil@latest

# 生成JUnit格式报告
go test -v ./... | go2xunit -output report.xml

# 生成HTML覆盖率报告
gocov test ./... | gocov-html > coverage.html

# 合并覆盖率报告
gocoverutil merge coverage1.out coverage2.out > merged-coverage.out
go tool cover -html=merged-coverage.out -o merged-coverage.html
```

## 9. 测试策略与最佳实践

### 9.1 测试金字塔实现
```bash
# 项目测试结构
project/
├── unit/               # 单元测试 (70%)
│   ├── service/
│   ├── repository/
│   └── utils/
├── integration/        # 集成测试 (20%)
│   ├── api/
│   └── db/
└── e2e/                # 端到端测试 (10%)
    ├── web/
    └── mobile/
```

### 9.2 测试标签策略
```go
// +build unit

package service

import "testing"

func TestUnitFunction(t *testing.T) {
    // 纯单元测试逻辑
}
```

```go
// +build integration

package db

import "testing"

func TestDBIntegration(t *testing.T) {
    t.Skip("This is an integration test")
    // 集成测试逻辑
}
```

```bash
# 运行特定标签的测试
go test -tags=unit ./...
go test -tags=integration ./...
go test -tags="unit integration" ./...
```
