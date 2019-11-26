# Nginx参数

官方文档参考：<http://nginx.org/en/docs/http/load_balancing.html>  
负载均衡算法参考：<https://github.com/CyC2018/CS-Notes/blob/master/notes/集群.md#一负载均衡>

## 常用的负载均衡算法

| 算法名称                                  | 描述                                                                                                                                                                            | Nginx是否支持            |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| 轮询（Round Robin）                       | 轮询算法把每个请求依次轮流发送到每个服务器上                                                                                                                                    | 是  （round-robin）【默认】      |
| 加权轮询（Weighted Round Robbin）         | 加权轮询是在轮询的基础上，根据服务器的性能差异，为服务器赋予一定的权值，性能高的服务器分配更高的权值。                                                                          | 否                       |
| 最少连接（least Connections）             | 最少连接算法就是将请求发送给当前最少连接数的服务器上。                                                                                                                          | 是   （least-connected） |
| 加权最少连接（Weighted Least Connection） | 在最少连接的基础上，根据服务器的性能为每台服务器分配权重，再根据权重计算出每台服务器能处理的连接数。                                                                            | 否                       |
| 随机算法（Random）                        | 把请求随机发送到服务器上。                                                                                                                                                      | 否                       |
| 源地址哈希法 (IP Hash)                    | 源地址哈希通过对客户端 IP 计算哈希值之后，再对服务器数量取模得到目标服务器的序号。<br/>可以保证同一 IP 的客户端的请求会转发到同一台服务器上，用来实现会话粘滞（Sticky Session） | 是 （ip-hash）           |

## 轮询

```nginx
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
```

> 以上示例是三台服务器将80端口的请求以负载均衡的方式提供服务。当负载均衡方式未被显示定义的时，使用的负载均衡算法是轮询`round-robin`，所有的请求都被代理到http协议模式下的`myapp1`分组下的服务器进行请求分发。  


> 为了负载均衡HTTPS而不是HTTP，只需要“https”作为协议  
> Nginx支持的负载协议包括`HTTP`, `HTTPS`, `FastCGI`, `uwsgi`, `SCGI`, `memcached`, `gRPC`。  
> 为了设置负载均衡协议到`FastCGI`, `uwsgi`, `SCGI`, `memcached`, `gRPC`需要使用参数`fastcgi_pass`, `uwsgi_pass`, `scgi_pass`, `memcached_pass`,`grpc_pass`  

## 最少连接

```nginx
  upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
```

## 源地址哈希法(会话持久化)

```nginx
upstream myapp1 {
    ip_hash;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```

## 加权 

```nginx
upstream myapp1 {
    server srv1.example.com weight=3;
    server srv2.example.com;
    server srv3.example.com;
}
```