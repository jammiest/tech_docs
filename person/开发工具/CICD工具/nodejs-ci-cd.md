# Node.js 项目 CI/CD

## 1. 概述

Node.js 项目的 CI/CD 流程需要处理 npm/yarn 依赖管理、JavaScript/TypeScript 编译、测试框架和部署策略。本指南提供完整的 Node.js 项目 CI/CD 实施方案，涵盖代码检查、测试、构建、容器化和部署全流程。

## 2. 环境配置

### 2.1 基础环境设置
```bash
#!/bin/bash
# setup-node-environment.sh

# 安装 Node.js 和 npm
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# 安装 yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install -y yarn

# 安装常用工具
sudo npm install -g npm@latest
sudo npm install -g typescript
sudo npm install -g ts-node
sudo npm install -g nodemon

# 安装版本管理工具
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 配置环境变量
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.bashrc

# 安装 Node.js 版本
nvm install 16
nvm install 18
nvm use 18

# 验证安装
node --version
npm --version
yarn --version
```

### 2.2 Docker 化 Node.js 环境
```dockerfile
# Dockerfile
# 构建阶段
FROM node:18-alpine AS builder

WORKDIR /app

# 安装系统依赖
RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    git

# 复制依赖文件
COPY package*.json ./
COPY yarn.lock ./

# 安装依赖
RUN npm ci --only=production

# 复制应用代码
COPY . .

# 构建应用（如果是 TypeScript）
RUN if [ -f tsconfig.json ]; then npm run build; fi

# 最终阶段
FROM node:18-alpine

WORKDIR /app

# 安装运行时依赖
RUN apk add --no-cache curl

# 从构建阶段复制已安装的依赖
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./

# 创建非root用户
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

# 设置环境变量
ENV NODE_ENV=production
ENV PORT=3000

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# 启动命令
CMD ["node", "dist/index.js"]
```

## 3. CI/CD 流水线设计

### 3.1 GitHub Actions 配置
```yaml
# .github/workflows/node-ci-cd.yml
name: Node.js CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  DOCKER_IMAGE: 'ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}'

jobs:
  code-quality:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: ['16', '18']
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run TypeScript compiler
        run: npm run type-check
      
      - name: Run Prettier
        run: npm run format:check

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
      
      - name: Setup Node.js ${{ env.NODE_VERSION }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ env.NODE_VERSION }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: node-app
          path: |
            dist/
            package.json
            node_modules/

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
image: node:18-alpine

stages:
  - code_quality
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"

before_script:
  - node --version
  - npm --version
  - npm ci

code_quality:
  stage: code_quality
  script:
    - npm run lint
    - npm run type-check
    - npm run format:check

test:
  stage: test
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_PASSWORD: "postgres"
    POSTGRES_DB: "testdb"
    REDIS_PASSWORD: "redis"
    NODE_ENV: "test"
  script:
    - npm test -- --coverage
  artifacts:
    paths:
      - coverage/
    expire_in: 1 week

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
      - package.json
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

### 4.1 ESLint 配置
```json
// .eslintrc.js
module.exports = {
  env: {
    es2021: true,
    node: true,
    jest: true,
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'prettier',
    'plugin:import/recommended',
    'plugin:import/typescript',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: './tsconfig.json',
  },
  plugins: [
    '@typescript-eslint',
    'import',
    'prettier',
  ],
  rules: {
    'prettier/prettier': 'error',
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'error',
    '@typescript-eslint/no-explicit-any': 'error',
    'import/order': [
      'error',
      {
        'groups': [
          'builtin',
          'external',
          'internal',
          'parent',
          'sibling',
          'index',
        ],
        'newlines-between': 'always',
        'alphabetize': {
          'order': 'asc',
          'caseInsensitive': true,
        },
      },
    ],
  },
  settings: {
    'import/resolver': {
      typescript: {
        project: './tsconfig.json',
      },
    },
  },
};
```

### 4.2 TypeScript 配置
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    "noEmitOnError": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "@test/*": ["test/*"]
    }
  },
  "include": [
    "src/**/*",
    "test/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "coverage"
  ]
}
```

### 4.3 Prettier 配置
```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "all",
  "printWidth": 80,
  "endOfLine": "lf",
  "arrowParens": "avoid",
  "bracketSpacing": true,
  "quoteProps": "as-needed"
}
```

## 5. 测试策略

### 5.1 单元测试最佳实践
```typescript
// test/services/userService.test.ts
import { jest } from '@jest/globals';
import { UserService } from '../../src/services/userService';
import { UserRepository } from '../../src/repositories/userRepository';
import { User } from '../../src/models/user';
import { UserNotFoundError, InvalidEmailError } from '../../src/errors';

describe('UserService', () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockUserRepository = {
      findById: jest.fn(),
      findByEmail: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    } as jest.Mocked<UserRepository>;

    userService = new UserService(mockUserRepository);
  });

  describe('getUserById', () => {
    it('should return user when found', async () => {
      // Arrange
      const userId = 1;
      const expectedUser: User = {
        id: userId,
        email: 'test@example.com',
        name: 'Test User',
        createdAt: new Date(),
        updatedAt: new Date(),
      };
      mockUserRepository.findById.mockResolvedValue(expectedUser);

      // Act
      const result = await userService.getUserById(userId);

      // Assert
      expect(result).toEqual(expectedUser);
      expect(mockUserRepository.findById).toHaveBeenCalledWith(userId);
    });

    it('should throw UserNotFoundError when user not found', async () => {
      // Arrange
      const userId = 999;
      mockUserRepository.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(userService.getUserById(userId))
        .rejects
        .toThrow(UserNotFoundError);
    });
  });

  describe('createUser', () => {
    it('should create user with valid email', async () => {
      // Arrange
      const userData = {
        email: 'valid@example.com',
        name: 'Test User',
      };
      const expectedUser: User = {
        id: 1,
        ...userData,
        createdAt: new Date(),
        updatedAt: new Date(),
      };
      mockUserRepository.create.mockResolvedValue(expectedUser);

      // Act
      const result = await userService.createUser(userData);

      // Assert
      expect(result).toEqual(expectedUser);
      expect(mockUserRepository.create).toHaveBeenCalledWith(userData);
    });

    it('should throw InvalidEmailError with invalid email', async () => {
      // Arrange
      const userData = {
        email: 'invalid-email',
        name: 'Test User',
      };

      // Act & Assert
      await expect(userService.createUser(userData))
        .rejects
        .toThrow(InvalidEmailError);
    });
  });

  describe('updateUser', () => {
    it('should update user when found', async () => {
      // Arrange
      const userId = 1;
      const updateData = { name: 'Updated Name' };
      const existingUser: User = {
        id: userId,
        email: 'test@example.com',
        name: 'Test User',
        createdAt: new Date(),
        updatedAt: new Date(),
      };
      const updatedUser: User = {
        ...existingUser,
        ...updateData,
        updatedAt: new Date(),
      };
      mockUserRepository.findById.mockResolvedValue(existingUser);
      mockUserRepository.update.mockResolvedValue(updatedUser);

      // Act
      const result = await userService.updateUser(userId, updateData);

      // Assert
      expect(result).toEqual(updatedUser);
      expect(mockUserRepository.update).toHaveBeenCalledWith(userId, updateData);
    });

    it('should throw UserNotFoundError when updating non-existent user', async () => {
      // Arrange
      const userId = 999;
      const updateData = { name: 'Updated Name' };
      mockUserRepository.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(userService.updateUser(userId, updateData))
        .rejects
        .toThrow(UserNotFoundError);
    });
  });
});
```

### 5.2 集成测试配置
```typescript
// test/integration/userApi.test.ts
import request from 'supertest';
import { Express } from 'express';
import { createApp } from '../../src/app';
import { Database } from '../../src/database';

describe('User API Integration Tests', () => {
  let app: Express;
  let database: Database;

  beforeAll(async () => {
    const { app: expressApp, db } = await createApp();
    app = expressApp;
    database = db;
  });

  afterAll(async () => {
    await database.close();
  });

  beforeEach(async () => {
    // 清空测试数据
    await database.user.deleteMany();
  });

  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        name: 'Test User',
      };

      // Act
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .set('Accept', 'application/json');

      // Assert
      expect(response.status).toBe(201);
      expect(response.body).toMatchObject({
        email: userData.email,
        name: userData.name,
      });
      expect(response.body.id).toBeDefined();
    });

    it('should return 400 for duplicate email', async () => {
      // Arrange
      const userData = {
        email: 'duplicate@example.com',
        name: 'Test User',
      };

      // 先创建用户
      await request(app)
        .post('/api/users')
        .send(userData)
        .set('Accept', 'application/json');

      // Act - 尝试创建重复用户
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .set('Accept', 'application/json');

      // Assert
      expect(response.status).toBe(400);
      expect(response.body.error).toBe('Email already exists');
    });

    it('should return 400 for invalid email', async () => {
      // Arrange
      const userData = {
        email: 'invalid-email',
        name: 'Test User',
      };

      // Act
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .set('Accept', 'application/json');

      // Assert
      expect(response.status).toBe(400);
      expect(response.body.error).toBe('Invalid email format');
    });
  });

  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      // Arrange
      const userData = {
        email: 'test@example.com',
        name: 'Test User',
      };

      // 先创建用户
      const createResponse = await request(app)
        .post('/api/users')
        .send(userData)
        .set('Accept', 'application/json');

      const userId = createResponse.body.id;

      // Act - 获取用户
      const response = await request(app)
        .get(`/api/users/${userId}`)
        .set('Accept', 'application/json');

      // Assert
      expect(response.status).toBe(200);
      expect(response.body).toMatchObject(userData);
    });

    it('should return 404 for non-existent user', async () => {
      // Act
      const response = await request(app)
        .get('/api/users/999')
        .set('Accept', 'application/json');

      // Assert
      expect(response.status).toBe(404);
      expect(response.body.error).toBe('User not found');
    });
  });

  describe('GET /api/users', () => {
    it('should return list of users', async () => {
      // Arrange - 创建多个用户
      const users = [
        { email: 'user1@example.com', name: 'User One' },
        { email: 'user2@example.com', name: 'User Two' },
      ];

      for (const user of users) {
        await request(app)
          .post('/api/users')
          .send(user)
          .set('Accept', 'application/json');
      }

      // Act
      const response = await request(app)
        .get('/api/users')
        .set('Accept', 'application/json');

      // Assert
      expect(response.status).toBe(200);
      expect(response.body).toHaveLength(users.length);
      expect(response.body).toEqual(
        expect.arrayContaining([
          expect.objectContaining(users[0]),
          expect.objectContaining(users[1]),
        ]),
      );
    });
  });
});
```

## 6. 部署策略

### 6.1 Kubernetes 部署配置
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
  namespace: production
  labels:
    app: node-app
    environment: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: node-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: node-app
        environment: production
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
    spec:
      containers:
      - name: node-app
        image: ghcr.io/myorg/node-app:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        env:
        - name: NODE_ENV
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
  name: node-app-service
  namespace: production
spec:
  selector:
    app: node-app
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
```

### 6.2 健康检查配置
```typescript
// src/health.ts
import { Router } from 'express';
import { Redis } from 'ioredis';
import { Database } from '../database';

const healthRouter = Router();

export interface HealthCheckResponse {
  status: 'healthy' | 'unhealthy';
  timestamp: string;
  services: {
    database: string;
    redis: string;
    [key: string]: string;
  };
}

export function createHealthRouter(database: Database, redis: Redis): Router {
  healthRouter.get('/health', async (req, res) => {
    const response: HealthCheckResponse = {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: {
        database: 'unknown',
        redis: 'unknown',
      },
    };

    try {
      // 检查数据库连接
      await database.$queryRaw`SELECT 1`;
      response.services.database = 'healthy';
    } catch (error) {
      response.status = 'unhealthy';
      response.services.database = `unhealthy: ${error.message}`;
    }

    try {
      // 检查Redis连接
      await redis.ping();
      response.services.redis = 'healthy';
    } catch (error) {
      response.status = 'unhealthy';
      response.services.redis = `unhealthy: ${error.message}`;
    }

    if (response.status === 'unhealthy') {
      return res.status(503).json(response);
    }

    res.json(response);
  });

  healthRouter.get('/health/liveness', (req, res) => {
    res.json({ status: 'alive' });
  });

  healthRouter.get('/health/readiness', async (req, res) => {
    try {
      await database.$queryRaw`SELECT 1`;
      res.json({ status: 'ready' });
    } catch (error) {
      res.status(503).json({ status: 'not ready' });
    }
  });

  return healthRouter;
}
```

## 7. 性能优化

### 7.1 PM2 集群配置
```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'node-app',
    script: './dist/index.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
    // 日志配置
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    out_file: '/var/log/node-app/out.log',
    error_file: '/var/log/node-app/error.log',
    combine_logs: true,
    // 性能调优
    max_memory_restart: '1G',
    watch: false,
    ignore_watch: ['node_modules', 'logs'],
    // 优雅关机
    kill_timeout: 3000,
    wait_ready: true,
    listen_timeout: 3000,
    // 重启策略
    autorestart: true,
    restart_delay: 3000,
    // 高级配置
    node_args: [
      '--max-old-space-size=1024',
      '--optimize-for-size',
      '--max-semi-space-size=128',
    ],
    interpreter_args: [],
  }],
};
```

### 7.2 内存优化配置
```typescript
// src/performance.ts
import cluster from 'cluster';
import os from 'os';

export class PerformanceOptimizer {
  static enableClusterMode(): void {
    if (cluster.isPrimary) {
      const numCPUs = os.cpus().length;
      console.log(`Master ${process.pid} is running`);
      console.log(`Forking ${numCPUs} workers...`);

      // 创建工作进程
      for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died. Restarting...`);
        cluster.fork();
      });
    } else {
      // 工作进程启动应用
      require('./server');
    }
  }

  static configureMemoryLimits(): void {
    // 设置内存限制
    const memoryLimit = process.env.NODE_MEMORY_LIMIT || '512';
    const maxOldSpaceSize = parseInt(memoryLimit, 10);
    
    if (global.gc) {
      // 在测试环境中启用手动GC
      setInterval(() => {
        global.gc!();
      }, 30000);
    }

    process.on('warning', (warning) => {
      if (warning.name === 'MaxListenersExceededWarning') {
        console.warn('MaxListenersExceededWarning:', warning.message);
      }
    });

    // 监控内存使用
    setInterval(() => {
      const memoryUsage = process.memoryUsage();
      const memoryUsageMB = {
        rss: Math.round(memoryUsage.rss / 1024 / 1024),
        heapTotal: Math.round(memoryUsage.heapTotal / 1024 / 1024),
        heapUsed: Math.round(memoryUsage.heapUsed / 1024 / 1024),
        external: Math.round(memoryUsage.external / 1024 / 1024),
      };

      if (memoryUsageMB.heapUsed > maxOldSpaceSize * 0.8) {
        console.warn('High memory usage:', memoryUsageMB);
      }
    }, 10000);
  }
}
```

## 8. 安全最佳实践

### 8.1 安全扫描集成
```yaml
# .github/workflows/security-scan.yml
name: Node.js Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'
  push:
    branches: [ main ]

jobs:
  npm-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Run npm audit
        run: npm audit --audit-level=high

  snyk-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t node-app:latest .
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'node-app:latest'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
          ignore-unfixed: true
```

### 8.2 安全中间件配置
```typescript
// src/security.ts
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import cors from 'cors';
import { Express } from 'express';

export function configureSecurity(app: Express): void {
  // Helmet - 安全头部
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
      },
    },
    crossOriginEmbedderPolicy: false,
  }));

  // CORS 配置
  app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowedHeaders: ['Content-Type', 'Authorization'],
  }));

  // 速率限制
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15分钟
    max: 100, // 每个IP限制100个请求
    message: 'Too many requests from this IP, please try again later.',
    standardHeaders: true,
    legacyHeaders: false,
  });

  app.use(limiter);

  // XSS 保护
  app.use((req, res, next) => {
    res.setHeader('X-XSS-Protection', '1; mode=block');
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    next();
  });

  // 防止参数污染
  app.use((req, res, next) => {
    if (req.query && typeof req.query === 'object') {
      const queryKeys = Object.keys(req.query);
      if (queryKeys.some(key => key.includes('[') || key.includes(']'))) {
        return res.status(400).json({ error: 'Invalid query parameters' });
      }
    }
    next();
  });
}
```
