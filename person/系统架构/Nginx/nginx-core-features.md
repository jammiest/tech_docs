# Nginx 核心功能体系

作为高性能的 Web 服务器和反向代理服务器，Nginx 提供了一系列强大的核心功能。以下是 Nginx 的主要功能分类及其技术实现：

## 1. Web 服务功能

### 1.1 静态资源服务
```nginx
location /static/ {
    root /data/www;
    expires 30d;
    access_log off;
}
```
- 支持 sendfile 零拷贝技术
- 可配置缓存控制头（ETag/Last-Modified）
- 自动索引生成（autoindex）

### 1.2 动态内容代理
```nginx
location /api/ {
    proxy_pass http://backend;
    proxy_set_header Host $host;
}
```

## 2. 反向代理功能

### 2.1 负载均衡
支持多种算法：
```nginx
upstream backend {
    least_conn;
    server 10.0.0.1:8080 weight=5;
    server 10.0.0.2:8080;
}
```
算法类型：
- 轮询（默认）
- 加权轮询
- 最少连接
- IP哈希
- 一致性哈希（需第三方模块）

### 2.2 健康检查
```nginx
server 10.0.0.1:8080 max_fails=3 fail_timeout=30s;
```
健康状态判断公式：

$$ \text{健康状态} = \begin{cases} 
\text{正常} & \text{当失败次数} < \text{max_fails} \\
\text{不可用} & \text{当持续超时} \geq \text{fail_timeout}
\end{cases} $$

## 3. 缓存功能

### 3.1 代理缓存
```nginx
proxy_cache_path /data/cache levels=1:2 keys_zone=mycache:10m inactive=60m;
proxy_cache_key "$scheme$request_method$host$request_uri";
```

缓存命中率计算：
$$ \text{Hit Ratio} = \frac{\text{Cache Hits}}{\text{Total Requests}} \times 100\% $$

### 3.2 FastCGI 缓存
```nginx
fastcgi_cache_path /var/cache/nginx levels=1:2 keys_zone=phpcache:100m;
```

## 4. 安全功能

### 4.1 访问控制
```nginx
location /admin/ {
    allow 192.168.1.0/24;
    deny all;
}
```

### 4.2 速率限制
```nginx
limit_req_zone $binary_remote_addr zone=reqlimit:10m rate=10r/s;
limit_conn_zone $binary_remote_addr zone=connlimit:10m;
```

限流算法：
- 漏桶算法（Leaky Bucket）
- 令牌桶算法（需第三方模块）

## 5. 协议支持

### 5.1 HTTP/HTTPS
```nginx
server {
    listen 443 ssl;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
}
```

### 5.2 HTTP/2
```nginx
listen 443 ssl http2;
```

### 5.3 WebSocket 代理
```nginx
location /ws/ {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

## 6. 内容处理功能

### 6.1 重写与重定向
```nginx
rewrite ^/old/(.*)$ /new/$1 permanent;
```

### 6.2 响应头处理
```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header Content-Security-Policy "default-src 'self'";
```

## 7. 日志功能

### 7.1 访问日志
```nginx
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent';
access_log /var/log/nginx/access.log main;
```

### 7.2 错误日志分级
```nginx
error_log /var/log/nginx/error.log warn;
```

## 8. 性能优化功能

### 8.1 连接优化
```nginx
keepalive_timeout 65;
keepalive_requests 100;
tcp_nodelay on;
```

### 8.2 文件传输优化
```nginx
sendfile on;
tcp_nopush on;
aio on;
```

## 9. 邮件代理功能

### 9.1 SMTP/POP3/IMAP 代理
```nginx
mail {
    server_name mail.example.com;
    auth_http localhost:9000/auth;
    proxy_pass_error_message on;
}
```

## 10. 模块化扩展功能

### 10.1 动态模块加载
```nginx
load_module modules/ngx_http_geoip_module.so;
```

### 10.2 第三方模块支持
常见模块：
- Lua 模块（OpenResty）
- Brotli 压缩
- Headers More 模块

## 技术指标对比

| 功能类别 | 典型性能指标 | 优化参数 |
|----------|--------------|----------|
| 静态服务 | 50,000+ RPS | sendfile, aio |
| 反向代理 | 30,000+ RPS | keepalive, buffer优化 |
| SSL/TLS | 5,000+ TPS | TLS 1.3, OCSP Stapling |

Nginx 通过这些核心功能的组合，能够构建高性能、高可用的 Web 服务体系。实际部署时应根据业务需求选择适当的功能组合，并通过性能测试验证配置效果。