# Nginx Web服务功能深度解析

Nginx作为高性能的Web服务器，提供了全面的静态资源服务和动态内容处理能力。以下是Nginx Web服务功能的详细解析：

## 一、静态资源服务

### 1.1 基础文件服务
```nginx
server {
    listen 80;
    root /var/www/html;
    
    location / {
        index index.html;
    }
}
```
- **sendfile优化**：零拷贝技术减少CPU开销
```nginx
sendfile on;
tcp_nopush on;  # 配合sendfile使用
```
- **文件缓存控制**：
```nginx
location ~* \.(jpg|png|css|js)$ {
    expires 30d;
    add_header Cache-Control "public";
}
```

### 1.2 高级文件服务特性
- **字节范围请求**：
```nginx
location /video {
    mp4;
    mp4_buffer_size 1m;
    mp4_max_buffer_size 5m;
}
```
- **文件列表展示**：
```nginx
location /download {
    autoindex on;
    autoindex_exact_size off;
    autoindex_localtime on;
}
```

## 二、动态内容处理

### 2.1 FastCGI代理
```nginx
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    
    # 性能优化参数
    fastcgi_buffer_size 128k;
    fastcgi_buffers 4 256k;
}
```

### 2.2 负载均衡
```nginx
upstream app_servers {
    least_conn;
    server 10.0.1.10:8000 weight=3;
    server 10.0.1.11:8000;
    server 10.0.1.12:8000 backup;
}

location /api {
    proxy_pass http://app_servers;
    proxy_set_header Host $host;
}
```

## 三、内容压缩

### 3.1 Gzip压缩
```nginx
gzip on;
gzip_types text/plain text/css application/json;
gzip_min_length 1024;
gzip_proxied any;
```

### 3.2 Brotli压缩（需模块支持）
```nginx
brotli on;
brotli_types text/plain text/css application/json;
brotli_min_length 1024;
```

## 四、访问控制

### 4.1 基础认证
```nginx
location /admin {
    auth_basic "Admin Area";
    auth_basic_user_file /etc/nginx/conf.d/htpasswd;
}
```

### 4.2 IP限制
```nginx
location /secure {
    allow 192.168.1.0/24;
    deny all;
}
```

## 五、日志记录

### 5.1 自定义日志格式
```nginx
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent"';
                
access_log /var/log/nginx/access.log main;
```

### 5.2 条件日志记录
```nginx
map $status $loggable {
    ~^[23]  0;
    default 1;
}

access_log /var/log/nginx/error_requests.log combined if=$loggable;
```

## 六、安全防护

### 6.1 基础安全头
```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
```

### 6.2 速率限制
```nginx
limit_req_zone $binary_remote_addr zone=api:10m rate=100r/s;

location /api/ {
    limit_req zone=api burst=200 nodelay;
    proxy_pass http://backend;
}
```

## 七、性能优化

### 7.1 连接优化
```nginx
keepalive_timeout 65;
keepalive_requests 1000;
reset_timedout_connection on;
```

### 7.2 缓存优化
```nginx
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m inactive=60m;

location / {
    proxy_cache mycache;
    proxy_cache_valid 200 302 10m;
    proxy_cache_use_stale error timeout updating;
}
```

## 八、高级功能

### 8.1 WebSocket支持
```nginx
location /ws/ {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass http://websocket_backend;
}
```

### 8.2 HTTP/2配置
```nginx
server {
    listen 443 ssl http2;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    http2_push_preload on;
}
```

## 技术指标对比

| 功能类别 | 静态资源QPS | 动态代理QPS | 优化建议 |
|----------|-------------|-------------|----------|
| 小文件(1KB) | 50,000+ | 30,000+ | 启用sendfile |
| 大文件(10MB) | 5,000+ | 3,000+ | 调整buffer大小 |
| 高并发连接 | 10,000+ | 5,000+ | 优化keepalive |

## 最佳实践建议

1. **静态资源分离**：将静态资源部署到独立域名/CDN
2. **动静分离**：动态请求和静态请求使用不同location处理
3. **分级缓存**：浏览器缓存 → Nginx缓存 → 后端缓存
4. **监控指标**：关注请求耗时、错误率、缓存命中率等关键指标

通过合理配置这些功能，Nginx可以高效处理各种Web服务场景，从简单的静态网站到复杂的动态应用都能提供优异的性能表现。