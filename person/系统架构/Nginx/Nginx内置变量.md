# Nginx 内置变量

Nginx 提供了丰富的内置变量，这些变量在配置文件中广泛用于条件判断、日志记录、请求重写等场景。下面将分类详细说明这些变量及其用法。

## 一、HTTP 请求相关变量

### 1.1 基础请求信息

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `$request` | 完整的原始请求行 | `"GET /index.html HTTP/1.1"` |
| `$request_method` | 请求方法 | `GET`, `POST` |
| `$request_uri` | 完整原始URI（含参数） | `/search?q=nginx` |
| `$uri` | 当前请求的标准化URI（不含参数） | `/search` |
| `$args` | 查询字符串（问号后内容） | `q=nginx` |
| `$query_string` | 同 `$args` | `q=nginx` |
| `$is_args` | 如果有参数则为"?"，否则为空 | `?` 或 `""` |

### 1.2 请求头信息

| 变量名 | 对应请求头 | 示例值 |
|--------|------------|--------|
| `$http_host` | Host头 | `example.com` |
| `$http_user_agent` | User-Agent头 | `Mozilla/5.0` |
| `$http_referer` | Referer头 | `https://google.com` |
| `$http_cookie` | Cookie头 | `sessionid=1234` |
| `$http_accept` | Accept头 | `text/html` |
| `$http_accept_language` | Accept-Language | `en-US,en` |
| `$http_[header_name]` | 任意请求头 | 将`-`转为`_` |

## 二、响应相关变量

### 2.1 响应状态

| 变量名 | 说明 | 取值范围 |
|--------|------|----------|
| `$status` | 响应状态码 | `200`, `404`, `500`等 |
| `$body_bytes_sent` | 发送给客户端的body字节数 | 数字 |
| `$bytes_sent` | 发送给客户端的总字节数 | 数字 |

### 2.2 响应头控制

```nginx
add_header X-Request-ID $request_id;
```

## 三、连接信息变量

### 3.1 客户端信息

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `$remote_addr` | 客户端IP地址 | `192.168.1.1` |
| `$remote_port` | 客户端端口 | `54321` |
| `$remote_user` | Basic认证用户名 | `admin` |

### 3.2 服务端信息

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `$server_addr` | 服务器接收请求的IP | `10.0.0.1` |
| `$server_port` | 服务器接收请求的端口 | `80`, `443` |
| `$server_name` | 匹配请求的server_name | `example.com` |
| `$server_protocol` | 请求协议版本 | `HTTP/1.1` |

## 四、时间相关变量

### 4.1 时间点变量

| 变量名 | 说明 | 格式 |
|--------|------|------|
| `$time_iso8601` | ISO8601格式时间 | `2023-08-20T14:31:12+08:00` |
| `$time_local` | 本地时间（日志格式） | `20/Aug/2023:14:31:12 +0800` |

### 4.2 耗时变量

```nginx
# 计算请求处理时间（秒）
set $request_time $msec - $time_iso8601;
```

## 五、特殊用途变量

### 5.1 请求标识

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `$request_id` | 唯一请求ID | `f78b9a8c7e6d5f4a3b2c1` |
| `$connection` | 连接序列号 | `123456` |
| `$connection_requests` | 当前连接上的请求数 | `1`, `2` |

### 5.2 代理相关

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `$proxy_host` | proxy_pass指定的主机名 | `backend.example.com` |
| `$proxy_port` | proxy_pass指定的端口 | `8080` |
| `$upstream_addr` | 上游服务器地址 | `10.0.0.1:8080` |

## 六、HTTPS/SSL 变量

### 6.1 SSL 连接信息

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `$https` | 如果是SSL连接则为"on" | `on` 或 `""` |
| `$ssl_protocol` | SSL协议版本 | `TLSv1.2` |
| `$ssl_cipher` | 使用的加密套件 | `ECDHE-RSA-AES256-GCM-SHA384` |

### 6.2 客户端证书信息

```nginx
if ($ssl_client_verify != SUCCESS) {
    return 403;
}
```

## 七、位置匹配变量

### 7.1 正则捕获变量

```nginx
location ~ /user/(\d+) {
    set $user_id $1;  # 捕获第一个分组
}
```

### 7.2 位置名称

```nginx
location @fallback {
    internal;
    return 404 "Not Found";
}
```

## 八、变量使用示例

### 8.1 日志格式定制

```nginx
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent" '
                'rt=$request_time ua="$upstream_addr"';
```

### 8.2 条件重定向

```nginx
if ($args ~* "utm_source=([^&]+)") {
    set $source $1;
    rewrite ^/(.*)$ /track/$source/$1? redirect;
}
```

### 8.3 动态代理

```nginx
location /api/ {
    set $backend "default";
    if ($http_x_api_version = "v2") {
        set $backend "v2";
    }
    proxy_pass http://$backend;
}
```

## 九、变量性能注意事项

1. **变量计算开销**：
   - 简单变量（如`$uri`）开销小
   - 复杂变量（如正则捕获）开销较大

2. **变量缓存**：
   ```nginx
   # 频繁使用的变量可缓存
   set $cached_var $http_user_agent;
   ```

3. **变量作用域**：
   - 大部分变量是请求级别的
   - 连接级别变量（如`$connection`）在keepalive连接中会保持

## 十、自定义变量

### 10.1 变量创建

```nginx
set $my_var "default_value";
```

### 10.2 变量运算

```nginx
set $sum $arg_a + $arg_b;
```

### 10.3 变量类型

Nginx变量本质都是字符串，但某些上下文会进行类型转换：
$$ \text{数值运算} = \begin{cases} 
\text{字符串转数字} & \text{当操作数为数字字符串} \\
0 & \text{其他情况}
\end{cases} $$

掌握这些内置变量可以极大增强Nginx配置的灵活性和功能性。建议在实际使用中结合具体业务需求选择合适的变量组合。