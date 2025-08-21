# Nginx参数

参考：<http://nginx.org/en/docs/http/ngx_http_core_module.html#var_https>

## 常用变量

| 变量名              | 说明                                                           | 备注 |
| ------------------- | -------------------------------------------------------------- | ---- |
| `$args`             | 这个变量等于请求行中的参数，同`$query_string`                  |      |
| `$content_length`   | 请求头中的Content-length字段。                                 |      |
| `$content_type`     | 请求头中的Content-Type字段。                                   |      |
| `$document_root`    | 当前请求在root指令中指定的值。                                 |      |
| `$host`             | 请求主机头字段，否则为服务器名称。                             |      |
| `$http_user_agent`  | 客户端agent信息                                                |      |
| `$http_cookie`      | 客户端cookie信息                                               |      |
| `$limit_rate`       | 这个变量可以限制连接速率。                                     |      |
| `$request_method`   | 客户端请求的动作，通常为GET或POST。                            |      |
| `$remote_addr`      | 客户端的IP地址。                                               |      |
| `$remote_port`      | 客户端的端口。                                                 |      |
| `$remote_user`      | 已经经过Auth Basic Module验证的用户名。                        |      |
| `$request_filename` | 当前请求的文件路径，由root或alias指令与URI请求生成。           |      |
| `$scheme`           | HTTP方法（如http，https）。                                    |      |
| `$server_protocol`  | 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。                     |      |
| `$server_addr`      | 服务器地址，在完成一次系统调用后可以确定这个值。               |      |
| `$server_name`      | 服务器名称。                                                   |      |
| `$server_port`      | 请求到达服务器的端口号。                                       |      |
| `$request_uri`      | 包含请求参数的原始URI，不包含主机名，如 ”/foo/bar.php?arg=baz” |      |
| `$uri`              | 不带请求参数的当前URI，`$uri`不包含主机名，如”/foo/bar.html”   |      |
| `$document_uri`     | 与`$uri`相同                                                   |      |

### Demo1

> Nginx配置如下

```nginx
location = /variable {
    add_header  Content-Type 'text/html; charset=utf-8';
    return 200 "
        args = $args<br />
        binary_remote_addr = $binary_remote_addr<br />
        body_bytes_sent = $body_bytes_sent<br />
        content_length = $content_length<br />
        content_type= $content_type<br />
        document_root = $document_root<br />
        document_uri = $document_uri<br />
        host = $host<br />
        hostname   = $hostname<br />
        http_user_agent = $http_user_agent<br />
        is_args = $is_args<br />
        limit_rate  = $limit_rate<br />
        nginx_version  = $nginx_version<br />
        query_string  = $query_string<br />
        remote_addr  = $remote_addr<br />
        remote_port  = $remote_port<br />
        request_filename = $request_filename<br />
        request_body  = $request_body<br />
        request_body_file  = $request_body_file<br />
        request_completion  = $request_completion<br />
        request_method  = $request_method<br />
        request_uri  = $request_uri<br />
        scheme  = $scheme<br />
        server_addr  = $server_addr<br />
        server_name  = $server_name<br />
        server_port  = $server_port<br />
        server_protocol  = $server_protocol<br />";
}
```

> 请求

`http://test.juo.local/variable?a=1&b=2&c=3`

> 返回

```txt
args = a=1&b=2&c=3
binary_remote_addr = �� �
body_bytes_sent = 0
content_length =
content_type=
document_root = /etc/nginx/html
document_uri = /variable
host = test.juo.local
hostname = 5f76c3670c3a
http_user_agent = Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36
is_args = ?
limit_rate = 0
nginx_version = 1.17.0
query_string = a=1&b=2&c=3
remote_addr = 192.168.10.244
remote_port = 53743
request_filename = /etc/nginx/html/variable
request_body =
request_body_file =
request_completion =
request_method = GET
request_uri = /variable?a=1&b=2&c=3
scheme = http
server_addr = 172.17.0.4
server_name = test.juo.local
server_port = 80
server_protocol = HTTP/1.1
```

## 自定义变量

### Demo2

> nginx配置

```nginx
location ~ /self-variable {
    set $path 'juo';
    return 200 $path;
}
```

> 请求

`GET http://test.juo.local/self-variable`

> 返回

```txt
juo
```

## 自动设置变量

### Demo3

> 配置

```nginx
location ~ /self-variable(/)(.*)$ {
     set $path 'juo';
     return 200 "path = $path , 1 = $1 , 2 = $2";
 }
 ```

 > 请求

`GET http://test.tyloafer.cn/self-variable/haha`

> 返回

```txt
 path = juo , 1 = / , 2 = haha
 ```

---

### Demo4

```nginx
location ~ /self-variable(/)(.*)$ {
    set $path 'juo';
    if ($request_uri ~ 'haha') {
        return 200 $1;
    }
 }
```

!> 这时候 在进行第一次location正则匹配是产生的`$1`， 在进行第二次 `$request_uri ~ 'haha'` 正则匹配的时候会被覆盖，所以这时候，返回的是空白的，并不是上面的 `/`