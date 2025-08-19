# Docker 简介

## 什么是 Docker？

Docker 是一个开源的**容器化平台**，用于开发、部署和运行应用程序。它通过容器（Container）技术，将应用程序及其依赖项打包在一个轻量级、可移植的虚拟环境中，实现"一次构建，到处运行"。

### 核心概念
- **镜像（Image）**：只读模板，包含运行应用程序所需的代码、库和配置。
- **容器（Container）**：镜像的运行实例，是一个隔离的进程空间。
- **Dockerfile**：文本文件，定义如何构建镜像。
- **Docker Hub**：官方镜像仓库，提供大量预构建镜像。

## 为什么要用 Docker？

### 1. 环境一致性
```plaintext
开发环境 -> 测试环境 -> 生产环境
```

- 解决"在我机器上能跑"的问题
- 确保全生命周期环境一致

### 2. 资源高效

| 特性         | 传统虚拟机   | Docker 容器 | 技术原理                 |
|--------------|-------------|-------------|--------------------------|
| **启动时间** | 1-5分钟     | 1-5秒       | 容器共享宿主OS内核       |
| **磁盘占用** | 20-50GB     | 50-500MB    | 分层存储+写时复制        |
| **性能损耗** | 5-15%       | <1%         | 无硬件虚拟化开销         |
| **安全性**   | 强隔离      | 依赖配置    | 命名空间+cgroups限制     |

### 3. 快速部署

```bash
# 从镜像运行容器
docker run -d -p 80:80 nginx

# 构建自定义镜像
docker build -t myapp .
```

### 4. 微服务支持

- 每个服务独立容器化
- 方便横向扩展
- 支持 Kubernetes 等编排工具

### 5. 跨平台支持

- 可在 Windows/macOS/Linux 运行
- 支持云平台（AWS/Azure/GCP）

## 典型应用场景

### 开发环境标准化

- 新成员快速搭建环境
- 多项目不同依赖隔离

### CI/CD 流水线

```mermaid
graph LR
A[代码提交] --> B[Docker构建]
B --> C[测试]
C --> D[部署]
```

### 混合云部署

- 相同镜像可部署到任意支持Docker的环境

### 遗留系统现代化

- 将传统应用容器化
- 逐步迁移到云原生架构

## 快速开始示例

### 安装 Docker: [官方安装指南](https://docs.docker.com/get-docker/)

### 运行第一个容器:

```bash
docker run hello-world
```

### 构建自定义应用:

```dockerfile
# Dockerfile 示例
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

## 学习资源

- [官方文档](Play with Docker（在线实验环境）)
- [Docker 从入门到实践](https://yeasy.gitbook.io/docker_practice/)
- [Play with Docker](https://labs.play-with-docker.com/)（在线实验环境）