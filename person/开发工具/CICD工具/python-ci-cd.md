# Python 项目 CI/CD

## 1. 概述

Python 项目的 CI/CD 流程需要处理虚拟环境、依赖管理、多版本兼容性和打包发布等特性。本指南提供完整的 Python 项目 CI/CD 实施方案，涵盖代码检查、测试、打包、容器化和部署全流程。

## 2. 环境配置

### 2.1 基础环境设置
```bash
#!/bin/bash
# setup-python-environment.sh

# 安装 Python 和 pip
sudo apt update
sudo apt install -y python3.10 python3.10-venv python3.10-dev python3-pip

# 安装常用工具
sudo apt install -y git curl wget make

# 安装 Python 版本管理工具
curl -fsSL https://pyenv.run | bash

# 配置环境变量
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc

# 安装 Python 版本
pyenv install 3.10.12
pyenv install 3.9.17
pyenv global 3.10.12

# 安装常用 Python 工具
pip install --upgrade pip
pip install virtualenv pipenv poetry

# 验证安装
python --version
pip --version
```

### 2.2 Docker 化 Python 环境
```dockerfile
# Dockerfile
# 构建阶段
FROM python:3.10-slim AS builder

WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    libffi-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 最终阶段
FROM python:3.10-slim

WORKDIR /app

# 安装运行时依赖
RUN apt-get update && apt-get install -y \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# 从构建阶段复制已安装的包
COPY --from=builder /root/.local /root/.local
COPY --from=builder /app /app

# 确保脚本可执行
RUN chmod +x /app/entrypoint.sh

# 创建非root用户
RUN groupadd -r pythonapp && useradd -r -g pythonapp pythonapp
USER pythonapp

# 设置环境变量
ENV PATH=/root/.local/bin:$PATH
ENV PYTHONPATH=/app

# 暴露端口
EXPOSE 8000

# 启动命令
ENTRYPOINT ["/app/entrypoint.sh"]
```

## 3. CI/CD 流水线设计

### 3.1 GitHub Actions 配置
```yaml
# .github/workflows/python-ci-cd.yml
name: Python CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  PYTHON_VERSION: '3.10'
  POETRY_VERSION: '1.6.1'
  DOCKER_IMAGE: 'ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}'

jobs:
  code-quality:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.9', '3.10']
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Install Poetry
        run: pip install poetry==${{ env.POETRY_VERSION }}
      
      - name: Install dependencies
        run: poetry install --with dev
      
      - name: Run black
        run: poetry run black --check .
      
      - name: Run isort
        run: poetry run isort --check-only .
      
      - name: Run flake8
        run: poetry run flake8 .
      
      - name: Run mypy
        run: poetry run mypy .

  test:
    runs-on: ubuntu-latest
    needs: code-quality
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: --health-cmd="pg_isready" --health-interval=10s --health-timeout=5s --health-retries=3
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: --health-cmd="redis-cli ping" --health-interval=10s --health-timeout=5s --health-retries=3
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python ${{ env.PYTHON_VERSION }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install Poetry
        run: pip install poetry==${{ env.POETRY_VERSION }}
      
      - name: Install dependencies
        run: poetry install --with dev,test
      
      - name: Run tests
        run: poetry run pytest --cov=src --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: coverage.xml

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install Poetry
        run: pip install poetry==${{ env.POETRY_VERSION }}
      
      - name: Build package
        run: poetry build
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: python-package
          path: dist/*

  docker:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      
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
          kubectl set image deployment/myapp myapp=${{ env.DOCKER_IMAGE }}:${{ github.sha }} -n production
          kubectl rollout status deployment/myapp -n production
```

### 3.2 GitLab CI 配置
```yaml
# .gitlab-ci.yml
image: python:3.10-slim

stages:
  - code_quality
  - test
  - build
  - deploy

variables:
  PYTHON_VERSION: "3.10"
  POETRY_VERSION: "1.6.1"

before_script:
  - python --version
  - pip install poetry==$POETRY_VERSION
  - poetry install --with dev

code_quality:
  stage: code_quality
  script:
    - poetry run black --check .
    - poetry run isort --check-only .
    - poetry run flake8 .
    - poetry run mypy .

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
    - poetry install --with dev,test
    - poetry run pytest --cov=src --cov-report=xml
  artifacts:
    paths:
      - coverage.xml
    expire_in: 1 week

build:
  stage: build
  script:
    - poetry build
  artifacts:
    paths:
      - dist/*
    expire_in: 1 week

docker-build:
  stage: build
  image: docker:20.10
  services:
    - docker:20.10-dind
  variables:
    DOCKER_BUILDKIT: 1
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
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
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/myapp -n production
```

## 4. 代码质量工具

### 4.1 代码格式化配置
```ini
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py310']
include = '\.pyi?$'
exclude = '''
/(
    \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | _build
  | buck-out
  | build
  | dist
)/
'''

[tool.isort]
profile = "black"
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
line_length = 88
known_first_party = ["src"]

[tool.flake8]
max-line-length = 88
max-complexity = 10
ignore = "E203, E266, E501, W503"
exclude = [
    ".git",
    "__pycache__",
    "docs",
    "migrations",
    "static",
    "templates",
    "tests",
]

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_calls = true
```

### 4.2 预提交钩子配置
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-ast
      - id: check-json

  - repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
      - id: black
        args: [--line-length=88]

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort
        args: ["--profile", "black"]

  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
        additional_dependencies: [flake8-docstrings]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.4.1
    hooks:
      - id: mypy
        additional_dependencies: [types-requests, types-python-dateutil]

  - repo: https://github.com/pycqa/pydocstyle
    rev: 6.3.0
    hooks:
      - id: pydocstyle
        args: [--convention=pep257]

default_language_version:
  python: python3.10

default_stages: [commit]
```

## 5. 测试策略

### 5.1 单元测试最佳实践
```python
# tests/test_user_service.py
import pytest
from unittest.mock import Mock, patch
from datetime import datetime

from src.services.user_service import UserService
from src.models.user import User
from src.exceptions import UserNotFoundError, InvalidEmailError


class TestUserService:
    @pytest.fixture
    def mock_user_repository(self):
        return Mock()

    @pytest.fixture
    def user_service(self, mock_user_repository):
        return UserService(user_repository=mock_user_repository)

    def test_get_user_by_id_success(self, user_service, mock_user_repository):
        """测试成功获取用户"""
        # Arrange
        user_id = 1
        expected_user = User(id=user_id, email="test@example.com", name="Test User")
        mock_user_repository.get_by_id.return_value = expected_user

        # Act
        result = user_service.get_user_by_id(user_id)

        # Assert
        assert result == expected_user
        mock_user_repository.get_by_id.assert_called_once_with(user_id)

    def test_get_user_by_id_not_found(self, user_service, mock_user_repository):
        """测试用户不存在的情况"""
        # Arrange
        user_id = 999
        mock_user_repository.get_by_id.return_value = None

        # Act & Assert
        with pytest.raises(UserNotFoundError):
            user_service.get_user_by_id(user_id)

    def test_create_user_valid_email(self, user_service, mock_user_repository):
        """测试创建用户（有效邮箱）"""
        # Arrange
        email = "valid@example.com"
        name = "Test User"
        expected_user = User(id=1, email=email, name=name)
        mock_user_repository.create.return_value = expected_user

        # Act
        result = user_service.create_user(email, name)

        # Assert
        assert result == expected_user
        mock_user_repository.create.assert_called_once()

    @pytest.mark.parametrize("invalid_email", [
        "invalid",
        "invalid@",
        "invalid.com",
        "@example.com"
    ])
    def test_create_user_invalid_email(self, user_service, mock_user_repository, invalid_email):
        """测试创建用户（无效邮箱）"""
        # Act & Assert
        with pytest.raises(InvalidEmailError):
            user_service.create_user(invalid_email, "Test User")
        
        # 确保没有调用repository
        mock_user_repository.create.assert_not_called()

    @pytest.mark.asyncio
    async def test_async_operations(self, user_service, mock_user_repository):
        """测试异步操作"""
        # Arrange
        user_id = 1
        expected_user = User(id=user_id, email="test@example.com", name="Test User")
        mock_user_repository.get_by_id_async.return_value = expected_user

        # Act
        result = await user_service.get_user_by_id_async(user_id)

        # Assert
        assert result == expected_user
```

### 5.2 集成测试配置
```python
# tests/integration/test_user_api.py
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession

from src.main import app
from src.models.user import User
from src.database import get_db


@pytest.mark.asyncio
class TestUserAPI:
    @pytest.fixture(autouse=True)
    async def setup_db(self, async_db: AsyncSession):
        """测试数据准备"""
        # 清空表
        await async_db.execute("TRUNCATE TABLE users RESTART IDENTITY CASCADE")
        await async_db.commit()

        # 插入测试数据
        test_users = [
            User(email="user1@example.com", name="User One"),
            User(email="user2@example.com", name="User Two"),
        ]
        async_db.add_all(test_users)
        await async_db.commit()
        yield

    async def test_get_users(self, async_client: AsyncClient):
        """测试获取用户列表"""
        # Act
        response = await async_client.get("/api/users")
        
        # Assert
        assert response.status_code == 200
        data = response.json()
        assert len(data) == 2
        assert data[0]["email"] == "user1@example.com"

    async def test_create_user(self, async_client: AsyncClient):
        """测试创建用户"""
        # Arrange
        user_data = {
            "email": "new@example.com",
            "name": "New User"
        }

        # Act
        response = await async_client.post("/api/users", json=user_data)
        
        # Assert
        assert response.status_code == 201
        data = response.json()
        assert data["email"] == "new@example.com"
        assert data["name"] == "New User"

    async def test_create_user_duplicate_email(self, async_client: AsyncClient):
        """测试重复邮箱创建用户"""
        # Arrange
        user_data = {
            "email": "user1@example.com",  # 已存在的邮箱
            "name": "Duplicate User"
        }

        # Act
        response = await async_client.post("/api/users", json=user_data)
        
        # Assert
        assert response.status_code == 400
        assert "already exists" in response.json()["detail"]

    async def test_get_user_by_id(self, async_client: AsyncClient):
        """测试根据ID获取用户"""
        # Act
        response = await async_client.get("/api/users/1")
        
        # Assert
        assert response.status_code == 200
        data = response.json()
        assert data["id"] == 1
        assert data["email"] == "user1@example.com"

    async def test_get_user_not_found(self, async_client: AsyncClient):
        """测试获取不存在的用户"""
        # Act
        response = await async_client.get("/api/users/999")
        
        # Assert
        assert response.status_code == 404
```

## 6. 部署策略

### 6.1 Kubernetes 部署配置
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
  namespace: production
  labels:
    app: python-app
    environment: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: python-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: python-app
        environment: production
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8000"
    spec:
      containers:
      - name: python-app
        image: ghcr.io/myorg/python-app:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8000
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "250m"
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-secrets
              key: url
        envFrom:
        - configMapRef:
            name: app-config
---
apiVersion: v1
kind: Service
metadata:
  name: python-app-service
  namespace: production
spec:
  selector:
    app: python-app
  ports:
  - port: 80
    targetPort: 8000
  type: ClusterIP
```

### 6.2 健康检查配置
```python
# src/health.py
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from redis.asyncio import Redis
from datetime import datetime

from src.database import get_db
from src.redis import get_redis

router = APIRouter(tags=["health"])


@router.get("/health")
async def health_check(
    db: AsyncSession = Depends(get_db),
    redis: Redis = Depends(get_redis)
):
    """健康检查端点"""
    checks = {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat(),
        "services": {}
    }

    # 检查数据库连接
    try:
        await db.execute("SELECT 1")
        checks["services"]["database"] = "healthy"
    except Exception as e:
        checks["status"] = "unhealthy"
        checks["services"]["database"] = f"unhealthy: {str(e)}"

    # 检查Redis连接
    try:
        await redis.ping()
        checks["services"]["redis"] = "healthy"
    except Exception as e:
        checks["status"] = "unhealthy"
        checks["services"]["redis"] = f"unhealthy: {str(e)}"

    if checks["status"] == "unhealthy":
        return JSONResponse(
            status_code=503,
            content=checks
        )
    
    return checks


@router.get("/health/liveness")
async def liveness_probe():
    """存活探针"""
    return {"status": "alive"}


@router.get("/health/readiness")
async def readiness_probe(db: AsyncSession = Depends(get_db)):
    """就绪探针"""
    try:
        await db.execute("SELECT 1")
        return {"status": "ready"}
    except Exception:
        return JSONResponse(
            status_code=503,
            content={"status": "not ready"}
        )
```

## 7. 性能优化

### 7.1 Gunicorn 配置
```python
# gunicorn.conf.py
import multiprocessing

# 工作进程数
workers = multiprocessing.cpu_count() * 2 + 1

# 工作模式
worker_class = "uvicorn.workers.UvicornWorker"

# 绑定地址
bind = "0.0.0.0:8000"

# 日志配置
accesslog = "-"
errorlog = "-"
loglevel = "info"

# 进程名称
proc_name = "python-app"

# 超时设置
timeout = 30
keepalive = 2

# 最大请求数（防止内存泄漏）
max_requests = 1000
max_requests_jitter = 100

# 安全限制
limit_request_line = 4094
limit_request_fields = 100
limit_request_field_size = 8190
```

### 7.2 异步优化配置
```python
# src/optimization.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker
import aioredis

# 数据库连接池配置
DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/dbname"

engine = create_async_engine(
    DATABASE_URL,
    echo=False,
    pool_size=20,
    max_overflow=10,
    pool_timeout=30,
    pool_recycle=1800,
)

AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

# Redis连接池配置
redis_pool = aioredis.ConnectionPool.from_url(
    "redis://localhost:6379",
    max_connections=20,
    decode_responses=True
)

@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期管理"""
    # 启动时初始化
    await initialize_database()
    await initialize_redis()
    
    yield
    
    # 关闭时清理
    await close_database()
    await close_redis()

async def get_db() -> AsyncSession:
    """获取数据库会话"""
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

async def get_redis() -> aioredis.Redis:
    """获取Redis连接"""
    return aioredis.Redis(connection_pool=redis_pool)
```

## 8. 安全最佳实践

### 8.1 安全扫描集成
```yaml
# .github/workflows/security-scan.yml
name: Python Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'
  push:
    branches: [ main ]

jobs:
  bandit-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Run Bandit security scan
        run: |
          pip install bandit
          bandit -r src -f html -o bandit-report.html
      
      - name: Upload security report
        uses: actions/upload-artifact@v3
        with:
          name: bandit-report
          path: bandit-report.html

  safety-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Run Safety dependency check
        run: |
          pip install safety
          safety check --full-report

  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t python-app:latest .
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'python-app:latest'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
          ignore-unfixed: true
```

### 8.2 安全加固配置
```python
# src/security.py
from fastapi import Security, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
from typing import Optional

# 安全配置
SECRET_KEY = "your-secret-key"  # 从环境变量获取
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """生成密码哈希"""
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    """创建访问令牌"""
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(credentials: HTTPAuthorizationCredentials = Security(security)) -> dict:
    """验证令牌"""
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        raise HTTPException(
            status_code=401,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
```
