# Nginx 路由机制详解

Nginx 的路由系统是其核心功能之一，它通过多种指令组合实现灵活的请求分发。下面将详细介绍 Nginx 的路由机制。

## 1. 基础路由组件

### 1.1 server 块

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    # 路由规则
    location / {
        root /var/www/html;
    }
}
```

### 1.2 location 块

如前所述，location 是 Nginx 路由的核心指令，支持多种匹配模式。

## 2. 路由匹配流程

Nginx 的路由决策流程如下：

1. **监听端口**：根据 `listen` 指令确定接收请求的端口
2. **主机名匹配**：根据 `server_name` 匹配虚拟主机
3. **URI 路由**：在匹配的 server 块中，按优先级检查 location 规则
4. **请求处理**：执行匹配 location 中的指令集

## 3. 高级路由技术

### 3.1 条件路由

使用 `if` 指令实现条件路由（谨慎使用，可能影响性能）：

```nginx
location / {
    if ($http_user_agent ~* "mobile") {
        root /var/www/mobile;
    }
    
    if ($args ~* "debug=true") {
        add_header X-Debug-Mode "enabled";
    }
}
```

### 3.2 正则捕获路由

```nginx
location ~ ^/user/(\d+)/profile$ {
    # $1 捕获第一个分组
    proxy_pass http://user_service/$1;
}
```

### 3.3 基于变量的路由

```nginx
map $http_x_api_version $backend {
    default        "v1";
    "v2"           "v2";
    "beta"         "beta";
}

upstream v1 { server 10.0.0.1; }
upstream v2 { server 10.0.0.2; }
upstream beta { server 10.0.0.3; }

location /api {
    proxy_pass http://$backend;
}
```

## 4. 路由重写

### 4.1 rewrite 指令

```nginx
location /old {
    rewrite ^/old/(.*)$ /new/$1 permanent;
}
```

重写规则优先级：
$$ \text{执行顺序} = \begin{cases} 
1 & \text{server 级 rewrite} \\
2 & \text{location 级 rewrite} \\
3 & \text{location 内部处理} 
\end{cases} $$

### 4.2 try_files 指令

```nginx
location / {
    try_files $uri $uri/ @fallback;
}

location @fallback {
    proxy_pass http://backend;
}
```

## 5. 路由性能优化

### 5.1 匹配效率对比

| 匹配类型 | 时间复杂度 | 适用场景 |
|---------|------------|----------|
| 精确匹配 | O(1) | 高频端点 |
| 前缀匹配 | O(n) | 静态资源 |
| 正则匹配 | O(m) | 复杂规则 |

### 5.2 路由缓存

```nginx
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m;

location / {
    proxy_cache my_cache;
    proxy_pass http://backend;
}
```

缓存命中率计算：
$$ \text{命中率} = \frac{\text{缓存响应数}}{\text{总请求数}} \times 100\% $$

## 6. 微服务路由示例

```nginx
# API 网关路由
location ~ ^/api/(auth|order|payment)/(.*)$ {
    set $service $1;
    rewrite ^/api/[^/]+/(.*)$ /$1 break;
    proxy_pass http://$service_service;
}

# 前端路由处理
location / {
    try_files $uri $uri/ /index.html;
}
```

## 7. 调试技巧

### 7.1 路由调试日志

```nginx
log_format routing_debug '$remote_addr - $request [$location]';
server {
    set $location "init";
    
    location / {
        set $location "main";
        access_log /var/log/nginx/routing.log routing_debug;
    }
}
```

### 7.2 变量检查

```nginx
location /debug {
    add_header X-Route-Match $uri;
    add_header X-Request-Method $request_method;
    return 200 "Debug Info";
}
```

Nginx 的路由系统提供了强大的灵活性，合理设计路由规则可以显著提升系统性能和可维护性。建议根据实际业务需求选择适当的匹配策略和路由方式。