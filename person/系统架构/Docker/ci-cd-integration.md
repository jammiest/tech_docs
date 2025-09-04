# Docker CI/CD 集成完全指南

## CI/CD 流水线架构

```
+----------------+     +----------------+     +----------------+
|   代码提交      | --> |   持续集成     | --> |   持续部署     |
| - Git Hook    |     | - 构建测试     |     | - 环境部署    |
| - Webhook     |     | - 镜像构建     |     | - 滚动更新    |
| - PR触发       |     | - 安全扫描     |     | - 监控验证    |
+----------------+     +----------------+     +----------------+
```

## GitHub Actions 集成

### 基础 CI 工作流
```yaml
# .github/workflows/ci.yml
name: Docker CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Cache Docker layers
      uses: actions/cache@v3
      with:
        path: /tmp/.buildx-cache
        key: ${{ runner.os }}-buildx-${{ github.sha }}
        restore-keys: |
          ${{ runner.os }}-buildx-

    - name: Run tests
      run: docker-compose -f docker-compose.ci.yml run --rm app npm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          ${{ secrets.DOCKERHUB_USERNAME }}/app:latest
          ${{ secrets.DOCKERHUB_USERNAME }}/app:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### 多环境部署工作流
```yaml
# .github/workflows/deploy.yml
name: Deploy to Environment

on:
  workflow_run:
    workflows: ["Docker CI"]
    types: [completed]
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Deploy to production
      uses: appleboy/ssh-action@v0.1.3
      with:
        host: ${{ secrets.PRODUCTION_HOST }}
        username: ${{ secrets.SSH_USERNAME }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd /opt/app
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/app:${{ github.sha }}
          docker-compose -f docker-compose.prod.yml up -d
          docker image prune -f
```

## GitLab CI/CD 集成

### .gitlab-ci.yml 配置
```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_HOST: tcp://docker:2375
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""

services:
  - docker:dind

test:
  stage: test
  image: docker:latest
  script:
    - docker-compose -f docker-compose.ci.yml run --rm app npm test

build:
  stage: build
  image: docker:latest
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest

deploy_production:
  stage: deploy
  environment: production
  only:
    - main
  script:
    - apt-get update && apt-get install -y ssh
    - echo "$SSH_PRIVATE_KEY" > key.pem
    - chmod 600 key.pem
    - ssh -i key.pem -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_HOST "
        docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY &&
        docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA &&
        docker-compose -f /opt/app/docker-compose.prod.yml up -d &&
        docker image prune -f"
```

## Jenkins Pipeline 集成

### Jenkinsfile 配置
```groovy
// Jenkinsfile
pipeline {
    agent {
        docker {
            image 'docker:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Test') {
            steps {
                sh 'docker-compose -f docker-compose.ci.yml run --rm app npm test'
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW
                docker build -t myapp:$BUILD_NUMBER .
                docker tag myapp:$BUILD_NUMBER myapp:latest
                '''
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'docker scan myapp:$BUILD_NUMBER'
            }
        }
        
        stage('Push') {
            steps {
                sh '''
                docker push myapp:$BUILD_NUMBER
                docker push myapp:latest
                '''
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['production-server-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no deploy@production-server "
                        docker pull myapp:$BUILD_NUMBER &&
                        docker-compose -f /opt/app/docker-compose.prod.yml up -d &&
                        docker image prune -f"
                    """
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker-compose -f docker-compose.ci.yml down'
        }
    }
}
```

## 安全扫描集成

### 漏洞扫描流程
```yaml
# 安全扫描步骤
- name: Security Scan
  run: |
    docker scan --version
    docker scan --file Dockerfile myapp:${{ github.sha }} --exclude-base
    
    # 使用 Trivy 进行深度扫描
    docker run --rm aquasec/trivy image myapp:${{ github.sha }}
    
    # 使用 Grype
    docker run --rm anchore/grype myapp:${{ github.sha }}
```

### 安全策略执行
```yaml
- name: Check Security Policy
  run: |
    # 使用 OPA 检查策略
    docker run --rm -v $(pwd)/policies:/policies openpolicyagent/conftest test \
      Dockerfile --policy /policies/docker-security.rego
    
    # 检查镜像签名
    docker trust inspect myapp:latest
```

## 多环境配置管理

### 环境特定的 compose 文件
```yaml
# docker-compose.ci.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: ci
    environment:
      - NODE_ENV=test
      - DATABASE_URL=postgresql://test:test@db:5432/test
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=test
      - POSTGRES_PASSWORD=test
      - POSTGRES_DB=test

  # 测试工具
  test-runner:
    image: myapp:ci
    command: npm run test:e2e
    depends_on:
      - app
      - db
```

### 环境变量管理
```bash
# 环境特定的 .env 文件
# .env.ci
NODE_ENV=test
DATABASE_URL=postgresql://test:test@db:5432/test
REDIS_URL=redis://redis:6379
LOG_LEVEL=debug

# .env.production
NODE_ENV=production
DATABASE_URL=postgresql://produser:prodpass@db:5432/prod
REDIS_URL=redis://redis:6379
LOG_LEVEL=info
```

## 构建优化策略

### 构建缓存优化
```yaml
# 使用 BuildKit 缓存
- name: Build with cache
  run: |
    docker build \
      --cache-from=type=gha,scope=ci \
      --cache-to=type=gha,scope=ci,mode=max \
      -t myapp:${{ github.sha }} .
```

### 多架构构建
```yaml
- name: Multi-arch Build
  run: |
    docker buildx create --use
    docker buildx build \
      --platform linux/amd64,linux/arm64 \
      -t myapp:multi-arch \
      --push .
```

## 部署策略

### 蓝绿部署
```yaml
# 蓝绿部署脚本
- name: Blue-Green Deployment
  run: |
    # 部署新版本（绿色环境）
    docker-compose -f docker-compose.prod.yml up -d --scale app=3
    
    # 健康检查
    until curl -f http://localhost:3000/health; do
      sleep 5
    done
    
    # 切换流量
    docker-compose -f docker-compose.prod.yml up -d --scale app=3
    
    # 清理旧版本
    docker image prune -f
```

### 金丝雀发布
```yaml
- name: Canary Deployment
  run: |
    # 部署金丝雀版本
    docker run -d --name app-canary -p 3001:3000 myapp:${{ github.sha }}
    
    # 监控金丝雀
    sleep 30
    curl -f http://localhost:3001/health
    
    # 如果健康，全面部署
    docker-compose -f docker-compose.prod.yml up -d
```

## 监控与回滚

### 部署验证
```yaml
- name: Deployment Verification
  run: |
    # 健康检查
    curl -f http://$PRODUCTION_HOST/health
    
    # 性能测试
    docker run --rm alpine ab -n 1000 -c 10 http://$PRODUCTION_HOST/
    
    # 日志检查
    ssh $PRODUCTION_HOST "docker logs app --tail=100"
```

### 自动回滚
```yaml
- name: Auto Rollback
  if: failure()
  run: |
    # 回滚到上一个版本
    ssh $PRODUCTION_HOST "
      docker tag myapp:previous myapp:latest
      docker-compose -f docker-compose.prod.yml up -d
    "
    
    # 发送告警
    curl -X POST -H "Content-Type: application/json" \
      -d '{"text":"Deployment failed, rolled back to previous version"}' \
      $SLACK_WEBHOOK_URL
```

## 通知与报告

### 构建通知
```yaml
- name: Send Notification
  if: always()
  run: |
    STATUS=${{ job.status }}
    MESSAGE="Build $STATUS: ${{ github.repository }}@${{ github.sha }}"
    
    # Slack 通知
    curl -X POST -H "Content-Type: application/json" \
      -d "{\"text\":\"$MESSAGE\"}" \
      $SLACK_WEBHOOK_URL
    
    # Email 通知
    echo "$MESSAGE" | mail -s "Build Status" team@example.com
```

### 测试报告
```yaml
- name: Upload Test Results
  uses: actions/upload-artifact@v3
  with:
    name: test-results
    path: test-results.xml

- name: Publish Test Report
  uses: dorny/test-reporter@v1
  with:
    name: Jest Tests
    path: test-results.xml
    reporter: jest-junit
```

## 基础设施即代码

### Terraform 集成
```hcl
# infrastructure/main.tf
resource "docker_container" "app" {
  name  = "myapp"
  image = docker_image.app.latest
  ports {
    internal = 3000
    external = 80
  }
  env = [
    "NODE_ENV=production",
    "DATABASE_URL=${var.database_url}"
  ]
}

resource "docker_image" "app" {
  name = "myapp:latest"
}
```

### Ansible 部署
```yaml
# deploy.yml
- name: Deploy Docker Application
  hosts: production
  tasks:
    - name: Pull latest image
      docker_image:
        name: myapp:latest
        source: pull
        
    - name: Run container
      docker_container:
        name: myapp
        image: myapp:latest
        ports:
          - "80:3000"
        env:
          NODE_ENV: production
        restart_policy: always
```

## 最佳实践

### CI/CD 检查清单
1. ✅ 自动化测试集成
2. ✅ 安全扫描流程
3. ✅ 多环境部署支持
4. ✅ 回滚机制
5. ✅ 监控和告警
6. ✅ 密钥安全管理
7. ✅ 构建缓存优化
8. ✅ 部署验证

### 性能指标
- **构建时间**: < 5分钟
- **测试覆盖率**: > 80%
- **部署频率**: 多次/天
- **恢复时间**: < 5分钟
- **失败率**: < 5%

> 提示：CI/CD 流水线应该快速反馈，任何失败都应该立即通知团队。

!> 重要：生产环境部署必须经过严格的测试和安全检查，确保稳定性。