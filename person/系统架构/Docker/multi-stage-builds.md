# Docker 多阶段构建深度指南

## 基础概念

### 什么是多阶段构建
多阶段构建允许在单个 Dockerfile 中使用多个 `FROM` 指令，每个 `FROM` 指令开始一个新的构建阶段。可以只复制前一阶段的必要文件到新阶段，最终得到一个精简的镜像。

### 与传统构建对比
```
传统构建:
+-------------------------------+
| 包含构建工具和依赖的庞大镜像  |
| 包含源代码和构建中间产物      |
+-------------------------------+

多阶段构建:
+-------------------------------+
| 仅包含运行时必需的最小文件    |
| 不包含构建工具和源代码        |
+-------------------------------+
```

## 基本语法

### 基础结构
```dockerfile
# 第一阶段：构建阶段
FROM [基础镜像] AS [阶段名称]
# 构建命令...

# 第二阶段：运行阶段
FROM [精简基础镜像]
# 从前一阶段复制文件
COPY --from=[阶段名称] [源路径] [目标路径]
# 运行时配置...
```

### 完整示例
```dockerfile
# 构建阶段
FROM golang:1.18 AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

# 运行阶段
FROM alpine:3.15
WORKDIR /app
COPY --from=builder /app/server .
CMD ["./server"]
```

## 进阶用法

### 多阶段命名
```dockerfile
FROM node:16 AS frontend-builder
WORKDIR /app
COPY frontend/ .
RUN npm install && npm run build

FROM golang:1.18 AS backend-builder
WORKDIR /app
COPY backend/ .
RUN go build -o api .

FROM alpine:3.15
WORKDIR /app
COPY --from=frontend-builder /app/dist ./static
COPY --from=backend-builder /app/api .
CMD ["./api"]
```

### 选择性复制
```dockerfile
FROM ubuntu AS downloader
RUN apt-get update && apt-get install -y wget
RUN wget https://example.com/large-file.tar.gz

FROM alpine
COPY --from=downloader /large-file.tar.gz /tmp/
RUN tar xzf /tmp/large-file.tar.gz -C /usr/local \
    && rm /tmp/large-file.tar.gz
```

### 使用外部镜像作为阶段
```dockerfile
FROM nginx:alpine AS nginx-base

FROM alpine
COPY --from=nginx-base /etc/nginx/nginx.conf /etc/nginx/
COPY --from=nginx-base /usr/share/nginx/html /var/www
```

## 实战案例

### Go 应用构建
```dockerfile
# 第一阶段：构建
FROM golang:1.18 AS builder

# 设置模块代理
ENV GOPROXY=https://goproxy.cn,direct

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# 第二阶段：运行
FROM alpine:3.15
RUN apk --no-cache add ca-certificates tzdata
WORKDIR /root/
COPY --from=builder /app/app .
COPY --from=builder /app/config.yaml .

# 设置时区
ENV TZ=Asia/Shanghai

EXPOSE 8080
CMD ["./app"]
```

### Node.js 应用构建
```dockerfile
# 第一阶段：安装依赖
FROM node:16 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm install --production

# 第二阶段：构建
FROM node:16 AS builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN npm run build

# 第三阶段：运行
FROM node:16-alpine
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

ENV NODE_ENV=production
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### Java 应用构建
```dockerfile
# 第一阶段：构建
FROM maven:3.8.4-openjdk-11 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package -DskipTests

# 第二阶段：运行
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=builder /app/target/myapp.jar .
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

### Python 应用构建
```dockerfile
# 第一阶段：构建
FROM python:3.9 AS builder
WORKDIR /app

# 创建虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安装依赖
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# 第二阶段：运行
FROM python:3.9-slim
WORKDIR /app

# 复制虚拟环境
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 复制应用代码
COPY . .

EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

## 高级技巧

### 构建参数传递
```dockerfile
# 构建阶段
FROM golang:1.18 AS builder
ARG BUILD_VERSION=unknown
WORKDIR /app
COPY . .
RUN go build -ldflags "-X main.version=$BUILD_VERSION" -o app .

# 运行阶段
FROM alpine:3.15
COPY --from=builder /app/app .
CMD ["./app"]
```

### 构建缓存优化
```dockerfile
FROM node:16 AS builder

# 先复制 package.json 单独安装依赖
WORKDIR /app
COPY package*.json ./
RUN npm install

# 然后复制其他文件
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

### 多架构构建
```dockerfile
# 第一阶段：构建
FROM --platform=$BUILDPLATFORM golang:1.18 AS builder
ARG TARGETARCH
WORKDIR /app
COPY . .
RUN GOARCH=$TARGETARCH go build -o app .

# 第二阶段：运行
FROM alpine:3.15
COPY --from=builder /app/app .
CMD ["./app"]
```

## 性能优化

### 层合并技巧
```dockerfile
FROM alpine AS downloader
# 合并多个RUN命令减少层数
RUN apk add --no-cache curl tar && \
    curl -L https://example.com/file.tar.gz -o file.tar.gz && \
    tar xzf file.tar.gz && \
    rm file.tar.gz && \
    apk del curl tar
```

### 最小化最终镜像
```dockerfile
FROM scratch AS final
COPY --from=builder /app/app /
CMD ["/app"]
```

### 使用 distroless 镜像
```dockerfile
FROM golang:1.18 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o app .

FROM gcr.io/distroless/static
COPY --from=builder /app/app .
CMD ["./app"]
```

## 常见问题

### 调试构建阶段
```bash
# 构建特定阶段
docker build --target builder -t myapp-builder .

# 检查构建阶段内容
docker run -it myapp-builder sh
```

### 缓存失效问题
```dockerfile
# 正确的缓存顺序
COPY package.json package-lock.json ./
RUN npm install
COPY . .
```

### 大文件处理
```dockerfile
# 使用单独的下载阶段
FROM alpine AS downloader
RUN apk add --no-cache wget && \
    wget https://example.com/large-file.zip && \
    unzip large-file.zip

FROM alpine
COPY --from=downloader /extracted-files /app
```

## 最佳实践

1. **命名阶段**：为每个阶段使用有意义的名称（AS builder）
2. **最小化最终镜像**：使用 alpine 或 scratch 作为最终阶段基础
3. **分离依赖安装**：先复制 package.json/go.mod 单独安装依赖
4. **清理构建产物**：在构建阶段完成后删除不必要的文件
5. **多架构支持**：使用 --platform 和 TARGETARCH 参数
6. **安全扫描**：对每个构建阶段进行安全扫描

> 提示：多阶段构建可以显著减小最终镜像大小，提高安全性，建议所有生产镜像都采用此方式构建。

!> 重要：确保最终镜像只包含运行应用所需的绝对最小内容，移除所有构建工具和中间文件。