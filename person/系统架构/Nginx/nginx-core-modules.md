# Nginx 核心模块深度解析

Nginx 的模块化架构是其高性能和灵活性的基础。以下是 Nginx 核心模块的全面技术解析：

## 一、核心模块分类

### 1.1 事件模块
```nginx
events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}
```
- **ngx_event_core_module**：基础事件处理
- **ngx_epoll_module**（Linux）：epoll 事件模型
- **ngx_kqueue_module**（BSD）：kqueue 事件模型

### 1.2 HTTP 核心模块
```nginx
http {
    include mime.types;
    default_type application/octet-stream;
    sendfile on;
}
```
- **ngx_http_core_module**：HTTP 基础功能
- **ngx_http_log_module**：访问日志记录

## 二、HTTP 处理流程模块

### 2.1 请求处理阶段模块
| 模块 | 处理阶段 | 功能 |
|------|----------|------|
| ngx_http_rewrite_module | SERVER_REWRITE/REWRITE | URL重写 |
| ngx_http_access_module | ACCESS | 访问控制 |
| ngx_http_proxy_module | CONTENT | 反向代理 |

### 2.2 11个处理阶段

$$
\begin{aligned}
&\text{请求处理流程:} \\
&\text{POST_READ} \rightarrow \text{SERVER_REWRITE} \rightarrow \text{FIND_CONFIG} \\
&\rightarrow \text{REWRITE} \rightarrow \text{POST_REWRITE} \rightarrow \text{PREACCESS} \\
&\rightarrow \text{ACCESS} \rightarrow \text{POST_ACCESS} \rightarrow \text{TRY_FILES} \\
&\rightarrow \text{CONTENT} \rightarrow \text{LOG}
\end{aligned}
$$

## 三、关键核心模块详解

### 3.1 ngx_http_core_module
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    
    location / {
        index index.html;
    }
}
```
- **核心指令**：
  - `listen`：监听端口
  - `server_name`：虚拟主机名
  - `root`：文档根目录
  - `location`：请求路由

### 3.2 ngx_http_proxy_module
```nginx
location / {
    proxy_pass http://backend;
    proxy_set_header Host $host;
    proxy_buffer_size 16k;
}
```
- **缓冲区配置**：
  ```nginx
  proxy_buffers 8 16k;
  proxy_busy_buffers_size 32k;
  ```

### 3.3 ngx_http_rewrite_module
```nginx
rewrite ^/old/(.*)$ /new/$1 last;
if ($http_user_agent ~* "mobile") {
    rewrite ^(.*)$ /mobile$1 break;
}
```
- **执行顺序**：
  $$
  \text{server rewrite} \rightarrow \text{location匹配} \rightarrow \text{location rewrite}
  $$

## 四、连接处理模块

### 4.1 ngx_http_upstream_module
```nginx
upstream backend {
    least_conn;
    server 10.0.1.1:8080 weight=3;
    server 10.0.1.2:8080;
    keepalive 32;
}
```
- **负载均衡算法**：
  - 轮询（默认）
  - 最少连接
  - IP哈希

### 4.2 ngx_http_ssl_module
```nginx
server {
    listen 443 ssl;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
}
```
- **性能优化**：
  ```nginx
  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout 10m;
  ```

## 五、内容处理模块

### 5.1 ngx_http_gzip_module
```nginx
gzip on;
gzip_types text/plain text/css application/json;
gzip_min_length 1024;
```
- **压缩效率**：
  $$
  \text{压缩比} = \frac{\text{原始大小} - \text{压缩后大小}}{\text{原始大小}} \times 100\%
  $$

### 5.2 ngx_http_sub_module
```nginx
location / {
    sub_filter 'old_text' 'new_text';
    sub_filter_once off;
}
```

## 六、访问控制模块

### 6.1 ngx_http_access_module
```nginx
location /admin {
    allow 192.168.1.0/24;
    deny all;
}
```

### 6.2 ngx_http_auth_basic_module
```nginx
location /secure {
    auth_basic "Restricted";
    auth_basic_user_file /etc/nginx/htpasswd;
}
```

## 七、日志模块

### 7.1 ngx_http_log_module
```nginx
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent';
access_log /var/log/nginx/access.log main;
```
- **日志缓存优化**：
  ```nginx
  access_log /var/log/nginx/access.log main buffer=32k flush=5m;
  ```

## 八、核心模块交互

### 8.1 模块执行顺序示例
```nginx
http {
    # ngx_http_core_module
    server {
        # ngx_http_rewrite_module
        rewrite ^/old /new;
        
        # ngx_http_proxy_module
        location / {
            proxy_pass http://backend;
        }
        
        # ngx_http_access_module
        location /admin {
            deny all;
        }
    }
}
```

## 九、性能关键模块

### 9.1 ngx_http_limit_req_module
```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
location /api {
    limit_req zone=one burst=20;
}
```
- **漏桶算法**：
  $$
  \text{请求处理速率} = \min(\text{到达速率}, \text{限制速率})
  $$

### 9.2 ngx_http_slice_module
```nginx
location / {
    slice 1m;
    proxy_cache_key $uri$slice_range;
    proxy_set_header Range $slice_range;
}
```

## 十、模块开发基础

### 10.1 模块基本结构
```c
// 模块定义
ngx_module_t ngx_http_example_module = {
    NGX_MODULE_V1,
    &ngx_http_example_module_ctx, // 模块上下文
    ngx_http_example_commands,    // 模块指令
    NGX_HTTP_MODULE,             // 模块类型
    NULL,                        // init master
    NULL,                        // init module
    NULL,                        // init process
    NULL,                        // init thread
    NULL,                        // exit thread
    NULL,                        // exit process
    NULL,                        // exit master
    NGX_MODULE_V1_PADDING
};
```

### 10.2 处理阶段挂载
```c
static ngx_int_t ngx_http_example_handler(ngx_http_request_t *r) {
    // 请求处理逻辑
    return NGX_OK;
}

static ngx_http_module_t ngx_http_example_module_ctx = {
    NULL,                                  // preconfiguration
    NULL,                                  // postconfiguration
    NULL,                                  // create main configuration
    NULL,                                  // init main configuration
    NULL,                                  // create server configuration
    NULL,                                  // merge server configuration
    NULL,                                  // create location configuration
    NULL                                   // merge location configuration
};

static ngx_command_t ngx_http_example_commands[] = {
    {
        ngx_string("example"),
        NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS,
        ngx_http_example,
        0,
        0,
        NULL
    },
    ngx_null_command
};
```

Nginx 的核心模块协同工作形成了其强大的 Web 服务能力。理解这些模块的功能和交互方式，对于高效配置和深度定制 Nginx 至关重要。建议通过分析 Nginx 源码和官方文档进一步掌握模块开发技术。