# Swoole 基础用法详解

Swoole 是一个高性能的 PHP 协程框架，下面将介绍其最基础和常用的功能用法。

## 1. 创建基础 TCP 服务器

```php
// 创建TCP服务器
$server = new Swoole\Server('0.0.0.0', 9501);

// 设置服务器参数
$server->set([
    'worker_num' => 4,      // 工作进程数量
    'max_request' => 1000,  // 每个worker最大处理请求数
]);

// 注册事件回调
$server->on('Connect', function ($server, $fd) {
    echo "客户端 {$fd} 已连接\n";
});

$server->on('Receive', function ($server, $fd, $reactorId, $data) {
    echo "收到来自 {$fd} 的数据: {$data}\n";
    $server->send($fd, "服务器回复: " . $data);
});

$server->on('Close', function ($server, $fd) {
    echo "客户端 {$fd} 已断开连接\n";
});

// 启动服务器
$server->start();
```

## 2. 创建 HTTP 服务器

```php
$http = new Swoole\Http\Server("0.0.0.0", 9501);

$http->on('Request', function ($request, $response) {
    // 获取请求信息
    $path = $request->server['request_uri'];
    $method = $request->server['request_method'];
    
    // 设置响应头
    $response->header('Content-Type', 'text/plain; charset=utf-8');
    
    // 响应内容
    $response->end("Hello Swoole\n请求路径: {$path}\n请求方法: {$method}");
});

$http->start();
```

## 3. WebSocket 服务器

```php
$ws = new Swoole\WebSocket\Server("0.0.0.0", 9502);

$ws->on('Open', function ($ws, $request) {
    echo "客户端 {$request->fd} 已连接\n";
});

$ws->on('Message', function ($ws, $frame) {
    echo "收到消息: {$frame->data}\n";
    $ws->push($frame->fd, "服务器收到: {$frame->data}");
});

$ws->on('Close', function ($ws, $fd) {
    echo "客户端 {$fd} 已断开\n";
});

$ws->start();
```

## 4. 协程基础用法

### 协程创建与执行

```php
Co\run(function () {
    // 创建协程
    go(function () {
        Co::sleep(1);
        echo "协程1执行完成\n";
    });
    
    go(function () {
        Co::sleep(0.5);
        echo "协程2执行完成\n";
    });
    
    echo "主协程继续执行\n";
});
```

### 协程HTTP客户端

```php
Co\run(function () {
    $cli = new Co\Http\Client('www.baidu.com', 80);
    $cli->setHeaders([
        'Host' => "www.baidu.com",
        'User-Agent' => 'Swoole/' . SWOOLE_VERSION
    ]);
    $cli->get('/');
    echo $cli->body;
    $cli->close();
});
```

## 5. 定时器使用

```php
// 一次性定时器
Swoole\Timer::after(2000, function () {
    echo "2秒后执行\n";
});

// 间隔定时器
$timerId = Swoole\Timer::tick(1000, function () {
    echo "每秒执行一次\n";
});

// 10秒后清除定时器
Swoole\Timer::after(10000, function () use ($timerId) {
    Swoole\Timer::clear($timerId);
    echo "定时器已清除\n";
});
```

## 6. 进程管理

```php
// 创建子进程
$process = new Swoole\Process(function (Swoole\Process $worker) {
    echo "子进程PID: " . $worker->pid . "\n";
    $worker->write("Hello Master\n");
    sleep(2);
});

// 启动进程
$process->start();

// 主进程读取子进程数据
echo "主进程收到: " . $process->read();

// 等待子进程结束
Swoole\Process::wait();
```

## 7. 基础配置参数

| 配置项 | 说明 | 默认值 | 建议值 |
|--------|------|--------|--------|
| worker_num | 工作进程数量 | 1 | CPU核数的1-4倍 |
| max_request | worker最大请求数 | 0 | 1000-10000 |
| dispatch_mode | 数据包分发策略 | 2 | 1或2 |
| daemonize | 是否守护进程化 | false | 生产环境true |
| log_file | 日志文件路径 | 无 | 生产环境必须设置 |
| open_tcp_nodelay | 禁用Nagle算法 | false | true |

## 8. 常见问题解决方案

1. **端口占用问题**：
   ```php
   $server->on('start', function ($server) {
       echo "服务已启动，监听端口: 9501\n";
   });
   ```

2. **进程异常退出**：
   ```php
   $server->set([
       'max_request' => 1000,  // 防止内存泄漏
       'reload_async' => true  // 安全重启
   ]);
   ```

3. **性能优化**：
   ```php
   $server->set([
       'open_tcp_nodelay' => true,
       'tcp_fastopen' => true
   ]);
   ```

这些基础用法涵盖了Swoole最常用的功能，掌握这些可以满足大部分开发需求。