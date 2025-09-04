# Nginx Rewrite 模块深度解析

Nginx 的 rewrite 模块（ngx_http_rewrite_module）是 URL 重写和重定向的核心模块，提供了强大的 URL 操作能力。以下从 10 个方面全面解析该模块：

## 一、模块基础

### 1.1 模块指令概览
```nginx
# 主要指令
rewrite regex replacement [flag];
return code [text|URL];
set $variable value;
if (condition) { ... }
```

### 1.2 执行阶段
rewrite 规则在以下阶段执行：
- **SERVER_REWRITE**：server 块中的 rewrite
- **REWRITE**：location 中的 rewrite
- **POST_REWRITE**：重写后的处理

## 二、rewrite 指令详解

### 2.1 基本语法
```nginx
rewrite ^/old/(.*)$ /new/$1 [flag];
```

### 2.2 标志位对比
| Flag     | 作用域       | 后续处理                          |
|----------|-------------|----------------------------------|
| last     | server      | 重新搜索 location                 |
| break    | location    | 停止 rewrite，继续其他指令         |
| redirect | 全局        | 302临时重定向                     |
| permanent| 全局        | 301永久重定向                     |

### 2.3 正则表达式
```nginx
rewrite "^(.*)/index\.html$" $1/ permanent;  # 去除index.html
```

## 三、if 条件指令

### 3.1 条件类型
```nginx
# 变量存在性
if ($http_referer) { ... }

# 字符串比较
if ($request_method = POST) { ... }

# 正则匹配
if ($host ~* "^example\.com$") { ... }

# 文件检查
if (-f $request_filename) { ... }
```

### 3.2 逻辑运算
```nginx
if ($http_user_agent ~* "mobile" -a $args ~ "id=\d+") {
    rewrite ^(.*)$ /mobile$1 break;
}
```

## 四、return 指令

### 4.1 状态码返回
```nginx
# 直接返回状态码
return 403;

# 返回带内容的响应
return 200 "OK";

# 重定向
return 301 https://$host$request_uri;
```

### 4.2 特殊状态码
```nginx
# Nginx特有：关闭连接
return 444;
```

## 五、set 指令

### 5.1 变量设置
```nginx
set $is_mobile 0;
if ($http_user_agent ~* "mobile") {
    set $is_mobile 1;
}
```

### 5.2 变量运算
```nginx
set $sum $arg_a + $arg_b;
set $ratio $bytes_sent / $content_length;
```

## 六、执行顺序规则

### 6.1 处理流程
$$
\begin{aligned}
&\text{请求到达} \rightarrow \text{server rewrite} \rightarrow \text{location匹配} \\
&\rightarrow \text{location rewrite} \rightarrow \text{其他处理}
\end{aligned}
$$

### 6.2 优先级示例
```nginx
server {
    rewrite ^/a /x;  # 规则1
    location /a {
        rewrite ^/a /y;  # 规则2
    }
}
```
执行顺序：规则1 → 规则2

## 七、正则表达式进阶

### 7.1 捕获组使用
```nginx
rewrite ^/product-(\d+)-(\d+).html$ /products?id=$1&cat=$2? last;
```

### 7.2 命名捕获组
```nginx
rewrite ^/user/(?<id>\d+)$ /profile?id=$id last;
```

## 八、性能优化

### 8.1 规则优化
1. **精确匹配优先**：
```nginx
location = /login { ... }
```

2. **减少回溯**：
```nginx
# 不好: rewrite ^/(.*)/(.*)/(.*)$ ...
# 更好: rewrite ^/([^/]+)/([^/]+)/([^/]+)$ ...
```

### 8.2 避免常见陷阱
1. **循环重定向**：
```nginx
# 错误示例
rewrite ^/(.*)$ /$1 permanent;

# 正确做法
rewrite ^/(.*)$ https://$host/$1 permanent;
```

2. **过度使用if**：
```nginx
# 避免
if ($uri ~* "\.jpg$") { ... }

# 推荐
location ~* \.jpg$ { ... }
```

## 九、调试技巧

### 9.1 重写日志
```nginx
rewrite_log on;
error_log /path/to/error.log notice;
```

### 9.2 变量追踪
```nginx
add_header X-Rewrite-Debug "$uri -> $request_uri";
```

## 十、实战案例

### 10.1 全站HTTPS
```nginx
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

### 10.2 旧URL迁移
```nginx
rewrite ^/old-page /new-page permanent;
```

### 10.3 多条件重写
```nginx
set $redirect 0;
if ($http_user_agent ~* "mobile") {
    set $redirect 1;
}
if ($args ~* "force_desktop=1") {
    set $redirect 0;
}
if ($redirect) {
    rewrite ^(.*)$ /mobile$1 break;
}
```

### 10.4 动态路由
```nginx
rewrite ^/shop/([^/]+)/([^/]+)/?$ /index.php?category=$1&product=$2? last;
```

通过深入理解 rewrite 模块，可以实现灵活的 URL 管理和 SEO 优化。关键点：
1. 理解不同 flag 的行为差异
2. 避免循环重定向
3. 合理使用正则表达式
4. 重视调试和日志分析
5. 考虑性能影响，优化规则顺序