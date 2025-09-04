# Swoole 介绍

Swoole 是一个面向生产环境的 PHP 异步网络通信引擎，使 PHP 开发人员可以编写高性能的异步并发 TCP、UDP、Unix Socket、HTTP，WebSocket 服务。

## 主要特性

1. **异步非阻塞**：基于事件驱动，支持高并发
2. **协程支持**：提供类似 Go 语言的协程编程模式
3. **高性能**：相比传统 PHP-FPM 模式性能提升显著
4. **多种协议支持**：TCP/UDP/HTTP/WebSocket 等

## 安装方式

```bash
pecl install swoole
```

或者在 Linux 系统上可以编译安装：

```bash
git clone https://github.com/swoole/swoole-src.git
cd swoole-src
phpize
./configure
make && make install
```

## 基本使用示例

### TCP 服务器

```php
$server = new Swoole\Server('0.0.0.0', 9501);

$server->on('Connect', function ($server, $fd) {
    echo "Client: Connect.\n";
});

$server->on('Receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, "Server: " . $data);
});

$server->on('Close', function ($server, $fd) {
    echo "Client: Close.\n";
});

$server->start();
```

### HTTP 服务器

```php
$http = new Swoole\Http\Server("0.0.0.0", 9501);

$http->on('Request', function ($request, $response) {
    $response->header("Content-Type", "text/plain");
    $response->end("Hello World\n");
});

$http->start();
```

## 协程示例

```php
Co\run(function() {
    $server = new Co\Http\Server("0.0.0.0", 9502, false);
    
    $server->handle('/', function ($request, $response) {
        $response->end("<h1>Index</h1>");
    });
    
    $server->handle('/test', function ($request, $response) {
        $response->end("<h1>Test</h1>");
    });
    
    $server->start();
});
```

## 性能对比

| 指标          | PHP-FPM       | Swoole        |
|---------------|--------------|--------------|
| 请求/秒(QPS)  | ~1,000       | ~50,000      |
| 内存占用      | 较高         | 较低         |
| 长连接支持    | 不支持       | 支持         |
| 协程支持      | 不支持       | 支持         |

## 适用场景

1. 高并发实时应用
2. 微服务架构
3. 游戏服务器
4. 物联网(IoT)应用
5. 实时通信系统

## 注意事项

1. Swoole 的某些功能需要 PHP 7.0 或更高版本
2. 在 CLI 模式下运行，不是传统的 PHP-FPM 模式
3. 需要理解异步编程模型
4. 某些 PHP 扩展可能与 Swoole 不兼容

Swoole 极大地扩展了 PHP 的能力边界，使其可以应用于更多高性能场景，是 PHP 生态中的重要技术革新。