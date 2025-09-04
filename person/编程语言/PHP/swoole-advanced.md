# Swoole 高级用法详解

Swoole 提供了许多高级特性，可以帮助开发者构建高性能、高并发的应用程序。下面将介绍 Swoole 的一些高级用法。

## 1. 协程高级用法

### 协程池实现

```php
Co\run(function () {
    $pool = new Co\Channel(10); // 协程池大小为10
    
    // 初始化协程池
    for ($i = 0; $i < 10; $i++) {
        $pool->push(new Co\Coroutine());
    }
    
    // 使用协程池执行任务
    for ($i = 0; $i < 100; $i++) {
        go(function () use ($pool, $i) {
            $coroutine = $pool->pop();
            // 执行任务
            Co::sleep(0.1);
            echo "Task {$i} completed\n";
            $pool->push($coroutine);
        });
    }
});
```

### 协程批量执行

```php
Co\run(function () {
    $tasks = [];
    
    for ($i = 0; $i < 10; $i++) {
        $tasks[] = go(function () use ($i) {
            Co::sleep(1);
            return "Result {$i}";
        });
    }
    
    // 等待所有协程完成
    $results = Co\batch($tasks);
    print_r($results);
});
```

## 2. 连接池实现

### MySQL 连接池

```php
class MySQLPool
{
    private $pool;
    
    public function __construct($size = 10) {
        $this->pool = new Co\Channel($size);
        
        for ($i = 0; $i < $size; $i++) {
            $mysql = new Swoole\Coroutine\MySQL();
            $mysql->connect([
                'host' => '127.0.0.1',
                'user' => 'root',
                'password' => 'password',
                'database' => 'test'
            ]);
            $this->pool->push($mysql);
        }
    }
    
    public function get() {
        return $this->pool->pop();
    }
    
    public function put($mysql) {
        $this->pool->push($mysql);
    }
}

Co\run(function () {
    $pool = new MySQLPool();
    
    go(function () use ($pool) {
        $mysql = $pool->get();
        $result = $mysql->query('SELECT * FROM users');
        $pool->put($mysql);
        print_r($result);
    });
});
```

## 3. 高级服务器配置

### 多端口监听

```php
$server = new Swoole\Server('0.0.0.0', 9501);

// 添加TCP监听端口
$port1 = $server->listen('0.0.0.0', 9502, SWOOLE_SOCK_TCP);
$port1->on('Receive', function ($serv, $fd, $reactor_id, $data) {
    $serv->send($fd, "Port 9502 received: {$data}");
});

// 添加UDP监听端口
$port2 = $server->listen('0.0.0.0', 9503, SWOOLE_SOCK_UDP);
$port2->on('Packet', function ($serv, $data, $clientInfo) {
    $serv->sendto($clientInfo['address'], $clientInfo['port'], "UDP received: {$data}");
});

$server->start();
```

### SSL/TLS 加密通信

```php
$server = new Swoole\WebSocket\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);

$server->set([
    'ssl_cert_file' => '/path/to/cert.pem',
    'ssl_key_file' => '/path/to/key.pem',
    'ssl_ciphers' => 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256',
    'ssl_verify_peer' => false
]);

$server->start();
```

## 4. 高级进程管理

### 进程间共享内存

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->set([
    'worker_num' => 4,
    'enable_coroutine' => true
]);

// 创建共享内存表
$table = new Swoole\Table(1024);
$table->column('count', Swoole\Table::TYPE_INT);
$table->create();

$server->on('WorkerStart', function ($serv, $workerId) use ($table) {
    if ($workerId == 0) {
        Swoole\Timer::tick(1000, function () use ($table) {
            echo "Total count: " . $table->get('total', 'count') . "\n";
        });
    }
});

$server->on('Receive', function ($serv, $fd, $reactorId, $data) use ($table) {
    $table->incr('total', 'count');
    $serv->send($fd, "Count: " . $table->get('total', 'count'));
});

$server->start();
```

### 自定义进程管理

```php
$pm = new Swoole\Process\Manager();

$pm->add(function (Swoole\Process\Pool $pool, $workerId) {
    echo "Worker {$workerId} started\n";
    while (true) {
        sleep(1);
    }
});

$pm->add(function (Swoole\Process\Pool $pool, $workerId) {
    echo "Task Worker {$workerId} started\n";
    while (true) {
        sleep(2);
    }
});

$pm->start();
```

## 5. 高级协程客户端

### 协程Redis集群

```php
Co\run(function () {
    $redis = new Swoole\Coroutine\Redis\Cluster(
        ['127.0.0.1:7000', '127.0.0.1:7001']
    );
    
    $redis->set('key', 'value');
    echo $redis->get('key');
});
```

### 协程HTTP2客户端

```php
Co\run(function () {
    $client = new Swoole\Coroutine\Http2\Client('127.0.0.1', 9501);
    $client->connect();
    
    $req = new Swoole\Http2\Request;
    $req->path = '/index.html';
    $req->headers = [
        'host' => 'localhost',
        'user-agent' => 'swoole-http2-client'
    ];
    
    $client->send($req);
    $response = $client->recv();
    echo $response->data;
});
```

## 6. 高级事件循环

### 自定义事件循环

```php
Co\run(function () {
    $event = new Co\Event();
    
    go(function () use ($event) {
        Co::sleep(1);
        $event->set('event1');
    });
    
    go(function () use ($event) {
        $event->wait('event1');
        echo "Event1 triggered\n";
    });
});
```

### 协程屏障(Barrier)

```php
Co\run(function () {
    $barrier = new Co\Barrier(3);
    
    for ($i = 0; $i < 3; $i++) {
        go(function () use ($barrier, $i) {
            Co::sleep(1);
            echo "Task {$i} completed\n";
            $barrier->wait();
        });
    }
    
    $barrier->wait();
    echo "All tasks completed\n";
});
```

## 7. 高级调试与监控

### 性能分析

```php
$server = new Swoole\Server('0.0.0.0', 9501);
$server->set([
    'enable_coroutine' => true,
    'trace_event_worker' => true,
    'trace_flags' => SWOOLE_TRACE_ALL
]);

$server->on('WorkerStart', function ($serv, $workerId) {
    Swoole\Timer::tick(1000, function () {
        $stats = Swoole\Coroutine::stats();
        print_r($stats);
    });
});

$server->start();
```

### 内存分析

```php
Co\run(function () {
    $memory = new Swoole\Coroutine\Memory();
    
    go(function () use ($memory) {
        $memory->alloc(1024 * 1024); // 分配1MB内存
        Co::sleep(1);
        $memory->free();
    });
    
    Swoole\Timer::tick(500, function () use ($memory) {
        $usage = $memory->usage();
        echo "Memory usage: {$usage}\n";
    });
});
```

这些高级用法展示了 Swoole 的强大功能，可以帮助开发者构建更加复杂和高性能的应用程序。