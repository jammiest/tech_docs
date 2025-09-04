# Swoole常见错误及解决方案

Swoole 是一个强大的 PHP 协程框架，但在使用过程中开发者经常会犯一些错误。下面将详细介绍这些常见错误及其解决方案。

## 1. 阻塞操作错误

### 错误示例：在协程中使用阻塞 I/O

```php
Co\run(function () {
    go(function () {
        // 错误：使用阻塞的文件操作
        $content = file_get_contents('large_file.txt');
        echo strlen($content);
    });
});
```

**解决方案**：使用协程版本的 I/O 操作

```php
Co\run(function () {
    go(function () {
        // 正确：使用协程文件操作
        $content = Co::readFile('large_file.txt');
        echo strlen($content);
    });
});
```

## 2. 全局变量污染

### 错误示例：在 Worker 进程中误用全局变量

```php
$globalCounter = 0;

$server = new Swoole\Server('0.0.0.0', 9501);
$server->on('Receive', function ($serv, $fd, $reactorId, $data) {
    global $globalCounter;
    $globalCounter++;
    $serv->send($fd, "Counter: {$globalCounter}");
});
```

**解决方案**：使用 Swoole\Table 或进程间通信

```php
$server = new Swoole\Server('0.0.0.0', 9501);

// 创建共享内存表
$table = new Swoole\Table(1024);
$table->column('counter', Swoole\Table::TYPE_INT);
$table->create();
$table->set('global', ['counter' => 0]);

$server->on('Receive', function ($serv, $fd, $reactorId, $data) use ($table) {
    $table->incr('global', 'counter');
    $counter = $table->get('global', 'counter');
    $serv->send($fd, "Counter: {$counter}");
});
```

## 3. 协程上下文混淆

### 错误示例：在协程间共享连接资源

```php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

Co\run(function () use ($redis) {
    go(function () use ($redis) {
        $redis->set('key1', 'value1'); // 可能被其他协程打断
    });
    
    go(function () use ($redis) {
        $redis->set('key2', 'value2'); // 可能覆盖上一个操作
    });
});
```

**解决方案**：每个协程使用独立的连接或连接池

```php
Co\run(function () {
    go(function () {
        $redis = new Swoole\Coroutine\Redis();
        $redis->connect('127.0.0.1', 6379);
        $redis->set('key1', 'value1');
    });
    
    go(function () {
        $redis = new Swoole\Coroutine\Redis();
        $redis->connect('127.0.0.1', 6379);
        $redis->set('key2', 'value2');
    });
});
```

## 4. 错误处理不当

### 错误示例：忽略协程中的异常

```php
Co\run(function () {
    go(function () {
        // 可能抛出异常的操作
        $result = 1 / 0;
    });
});
```

**解决方案**：正确捕获和处理异常

```php
Co\run(function () {
    go(function () {
        try {
            $result = 1 / 0;
        } catch (Throwable $e) {
            echo "Error: " . $e->getMessage() . "\n";
        }
    });
});
```

## 5. 内存泄漏问题

### 错误示例：在长生命周期中累积数据

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$messageLog = [];

$server->on('Receive', function ($serv, $fd, $reactorId, $data) use (&$messageLog) {
    $messageLog[] = $data; // 内存会无限增长
    $serv->send($fd, "Received");
});
```

**解决方案**：使用适当的数据结构或定期清理

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$messageLog = new Swoole\RingQueue(1000); // 固定大小队列

$server->on('Receive', function ($serv, $fd, $reactorId, $data) use ($messageLog) {
    $messageLog->push($data); // 自动淘汰旧数据
    $serv->send($fd, "Received");
});
```

## 6. 协程死锁

### 错误示例：协程间相互等待

```php
Co\run(function () {
    $chan1 = new Co\Channel(1);
    $chan2 = new Co\Channel(1);
    
    go(function () use ($chan1, $chan2) {
        $chan1->push('data');
        $data = $chan2->pop(); // 等待
    });
    
    go(function () use ($chan1, $chan2) {
        $chan2->push('data');
        $data = $chan1->pop(); // 等待
    });
});
```

**解决方案**：设计合理的协程通信机制

```php
Co\run(function () {
    $chan = new Co\Channel(1);
    
    go(function () use ($chan) {
        $chan->push('data1');
        echo "Coroutine 1 sent data\n";
    });
    
    go(function () use ($chan) {
        $data = $chan->pop();
        echo "Coroutine 2 received: {$data}\n";
        $chan->push('data2');
    });
});
```

## 7. 错误配置参数

### 错误示例：不合理的工作进程配置

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->set([
    'worker_num' => 100, // 远大于CPU核心数
    'max_request' => 0   // 不限制请求数
]);
```

**解决方案**：根据实际情况合理配置

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->set([
    'worker_num' => swoole_cpu_num() * 2, // 通常为CPU核心数的1-4倍
    'max_request' => 10000, // 防止内存泄漏
    'reload_async' => true  // 安全重启
]);
```

## 8. 忽略连接管理

### 错误示例：不处理连接关闭

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->on('Receive', function ($serv, $fd, $reactorId, $data) {
    $serv->send($fd, "Hello");
    // 忘记检查连接是否有效
});
```

**解决方案**：正确处理连接状态

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->on('Receive', function ($serv, $fd, $reactorId, $data) {
    if ($serv->exist($fd)) { // 检查连接是否有效
        $serv->send($fd, "Hello");
    }
});
```

## 9. 协程嵌套过深

### 错误示例：无限递归协程

```php
Co\run(function () {
    function recursiveCoroutine($n) {
        go(function () use ($n) {
            if ($n > 0) {
                recursiveCoroutine($n - 1);
            }
        });
    }
    recursiveCoroutine(1000); // 可能导致栈溢出
});
```

**解决方案**：限制协程嵌套深度或使用循环

```php
Co\run(function () {
    $chan = new Co\Channel(1);
    
    go(function () use ($chan) {
        for ($i = 1000; $i > 0; $i--) {
            // 使用循环代替递归
            Co::sleep(0.001);
        }
        $chan->push('done');
    });
    
    $chan->pop();
});
```

## 10. 忽略性能监控

### 错误示例：不监控服务状态

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->start(); // 没有监控机制
```

**解决方案**：添加监控和统计功能

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->set([
    'stats_file' => '/tmp/swoole_stats.log'
]);

$server->on('WorkerStart', function ($serv, $workerId) {
    if ($workerId == 0) {
        Swoole\Timer::tick(5000, function () use ($serv) {
            $stats = $serv->stats();
            file_put_contents(
                '/tmp/swoole_monitor.log',
                date('Y-m-d H:i:s') . " " . json_encode($stats) . "\n",
                FILE_APPEND
            );
        });
    }
});

$server->start();
```

通过避免这些常见错误，可以显著提高 Swoole 应用的稳定性、性能和可靠性。