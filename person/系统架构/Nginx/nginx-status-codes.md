# Nginx 状态码全面解析

Nginx 作为 Web 服务器和反向代理服务器，会生成或传递多种 HTTP 状态码。以下是 Nginx 相关的状态码分类说明及其处理机制。

## 一、成功类状态码 (2xx)

### 1.1 常规成功
| 状态码           | 说明       | 触发场景                   |
| ---------------- | ---------- | -------------------------- |
| `200 OK`         | 请求成功   | 静态文件服务、正常代理响应 |
| `204 No Content` | 无内容返回 | 空响应配置                 |

### 1.2 特殊成功
```nginx
location /created {
    return 201;
}
```
| 状态码                | 说明       | 触发场景       |
| --------------------- | ---------- | -------------- |
| `201 Created`         | 资源已创建 | REST API 设计  |
| `206 Partial Content` | 部分内容   | 大文件分块传输 |

## 二、重定向类状态码 (3xx)

### 2.1 永久重定向
```nginx
rewrite ^/old/(.*)$ /new/$1 permanent;
```
| 状态码                   | 说明                 | Nginx 实现方式   |
| ------------------------ | -------------------- | ---------------- |
| `301 Moved Permanently`  | 永久移动             | `permanent` 参数 |
| `308 Permanent Redirect` | 永久重定向(保留方法) | `return 308`     |

### 2.2 临时重定向
```nginx
rewrite ^/temp/(.*)$ /new/$1 redirect;
```
| 状态码                   | 说明                 | Nginx 实现方式  |
| ------------------------ | -------------------- | --------------- |
| `302 Found`              | 临时移动             | `redirect` 参数 |
| `303 See Other`          | 见其他位置           | `return 303`    |
| `307 Temporary Redirect` | 临时重定向(保留方法) | `return 307`    |

### 2.3 特殊重定向
| 状态码             | 说明   | 触发场景                    |
| ------------------ | ------ | --------------------------- |
| `304 Not Modified` | 未修改 | 条件请求(If-Modified-Since) |

## 三、客户端错误类状态码 (4xx)

### 3.1 访问控制
```nginx
location /private {
    deny all;
    return 403; # 默认行为
}
```
| 状态码             | 说明     | 典型触发条件    |
| ------------------ | -------- | --------------- |
| `400 Bad Request`  | 错误请求 | 畸形HTTP请求    |
| `401 Unauthorized` | 未认证   | auth_basic 模块 |
| `403 Forbidden`    | 禁止访问 | 权限拒绝        |
| `404 Not Found`    | 未找到   | 文件不存在      |

### 3.2 请求限制
```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
location / {
    limit_req zone=one burst=5;
}
```
| 状态码                  | 说明     | 相关配置       |
| ----------------------- | -------- | -------------- |
| `429 Too Many Requests` | 请求过多 | limit_req 模块 |

### 3.3 其他客户端错误
| 状态码                   | 说明       | 触发场景              |
| ------------------------ | ---------- | --------------------- |
| `405 Method Not Allowed` | 方法不允许 | 限制请求方法          |
| `408 Request Timeout`    | 请求超时   | client_header_timeout |
| `413 Payload Too Large`  | 负载过大   | client_max_body_size  |
| `414 URI Too Long`       | URI过长    | 大额GET请求           |

## 四、服务端错误类状态码 (5xx)

### 4.1 代理错误
```nginx
proxy_next_upstream error timeout invalid_header;
```
| 状态码                      | 说明       | 触发原因              |
| --------------------------- | ---------- | --------------------- |
| `500 Internal Server Error` | 服务器错误 | FastCGI/PHP 崩溃      |
| `502 Bad Gateway`           | 网关错误   | 后端服务不可达        |
| `503 Service Unavailable`   | 服务不可用 | 主动维护停机          |
| `504 Gateway Timeout`       | 网关超时   | proxy_connect_timeout |

### 4.2 资源限制
| 状态码                     | 说明         | 相关配置 |
| -------------------------- | ------------ | -------- |
| `507 Insufficient Storage` | 存储空间不足 | 磁盘写满 |

## 五、Nginx 扩展状态码

### 5.1 特殊状态码
| 状态码                         | 说明       | 使用场景                    |
| ------------------------------ | ---------- | --------------------------- |
| `444 No Response`              | 关闭连接   | 安全拦截                    |
| `494 Request Header Too Large` | 请求头过大 | large_client_header_buffers |

### 5.2 状态码变量
```nginx
log_format error_log '$status - $upstream_status - $status';
```
| 变量               | 说明           |
| ------------------ | -------------- |
| `$status`          | 最终响应状态码 |
| `$upstream_status` | 上游服务状态码 |

## 六、状态码处理策略

### 6.1 错误页面定制
```nginx
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;
```

### 6.2 状态码重定向
```nginx
location /legacy {
    return 301 /new-path;
}
```

### 6.3 状态码与变量组合
```nginx
location /api {
    proxy_intercept_errors on;
    error_page 404 = @fallback;
}
```

## 七、状态码产生机制

### 7.1 Nginx 生成状态码流程
```
请求到达 → 阶段处理 → 错误检查 → 生成状态码 → 响应构造
```

### 7.2 状态码优先级规则
$$ \text{最终状态码} = \begin{cases} 
\text{显式return} & \text{最高} \\
\text{模块生成} & \text{中间} \\
\text{默认值} & \text{最低}
\end{cases} $$

## 八、状态码监控与分析

### 8.1 日志分析示例
```bash
# 统计状态码分布
awk '{print $9}' access.log | sort | uniq -c | sort -rn
```

### 8.2 监控指标

| 指标名称 | 计算公式               | 健康阈值 |
| -------- | ---------------------- | -------- |
| 错误率   | `(4xx+5xx)/total*100%` | < 1%     |
| 5xx率    | `5xx/total*100%`       | < 0.1%   |

## 九、最佳实践

1. **合理使用重定向**：
   - 永久重定向用301/308
   - 临时重定向用302/307

2. **错误处理策略**：
   ```nginx
   proxy_intercept_errors on;
   error_page 404 =200 /empty.json;
   ```

3. **状态码调试技巧**：
   ```nginx
   add_header X-Status-Debug $status;
   ```

理解这些状态码的含义和产生机制，可以帮助您更好地配置Nginx和排查问题。实际应用中应根据业务需求设计合理的状态码返回策略。