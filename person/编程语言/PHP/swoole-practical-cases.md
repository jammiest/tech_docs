# Swoole 经典实战案例

Swoole 在实际项目中有许多经典应用场景，下面将介绍几个典型的实战案例，包括代码实现和架构设计。

## 1. 高性能 WebSocket 聊天室

### 架构设计

```
前端客户端 ↔ Swoole WebSocket服务器 ↔ Redis(存储在线用户和消息)
```

### 实现代码

```php
class ChatServer
{
    private $server;
    private $redis;
    
    public function __construct()
    {
        $this->server = new Swoole\WebSocket\Server('0.0.0.0', 9501);
        $this->server->set([
            'worker_num' => 4,
            'enable_coroutine' => true
        ]);
        
        $this->redis = new Swoole\Coroutine\Redis();
        $this->redis->connect('127.0.0.1', 6379);
        
        $this->setupEvents();
    }
    
    private function setupEvents()
    {
        $this->server->on('Open', function ($server, $request) {
            $userId = $request->get['user_id'];
            $this->redis->hSet('online_users', $request->fd, $userId);
            echo "用户 {$userId} 已连接 (FD: {$request->fd})\n";
        });
        
        $this->server->on('Message', function ($server, $frame) {
            $data = json_decode($frame->data, true);
            
            if ($data['type'] === 'private_msg') {
                // 私聊消息
                $targetFd = $this->redis->hGet('user_fd', $data['target_id']);
                if ($targetFd && $server->exist($targetFd)) {
                    $server->push($targetFd, json_encode([
                        'type' => 'private_msg',
                        'from' => $this->redis->hGet('online_users', $frame->fd),
                        'content' => $data['content']
                    ]));
                }
            } else {
                // 广播消息
                foreach ($this->redis->hGetAll('online_users') as $fd => $userId) {
                    if ($server->exist($fd) && $fd != $frame->fd) {
                        $server->push($fd, json_encode([
                            'type' => 'broadcast_msg',
                            'from' => $this->redis->hGet('online_users', $frame->fd),
                            'content' => $data['content']
                        ]));
                    }
                }
            }
        });
        
        $this->server->on('Close', function ($server, $fd) {
            $userId = $this->redis->hGet('online_users', $fd);
            $this->redis->hDel('online_users', $fd);
            echo "用户 {$userId} 已断开 (FD: {$fd})\n";
        });
    }
    
    public function start()
    {
        $this->server->start();
    }
}

$chatServer = new ChatServer();
$chatServer->start();
```

## 2. 实时数据监控系统

### 架构设计

```
数据采集端 → Swoole TCP服务器 → 数据处理Worker → WebSocket推送 → 前端展示
```

### 实现代码

```php
class MonitoringSystem
{
    private $tcpServer;
    private $webSocketServer;
    private $stats = [];
    
    public function __construct()
    {
        // TCP服务器接收数据
        $this->tcpServer = new Swoole\Server('0.0.0.0', 9502);
        
        // WebSocket服务器推送数据
        $this->webSocketServer = new Swoole\WebSocket\Server('0.0.0.0', 9503);
        
        $this->setupServers();
    }
    
    private function setupServers()
    {
        // 共享内存表存储统计数据
        $table = new Swoole\Table(1024);
        $table->column('value', Swoole\Table::TYPE_FLOAT);
        $table->column('timestamp', Swoole\Table::TYPE_INT);
        $table->create();
        
        // TCP服务器配置
        $this->tcpServer->set([
            'worker_num' => 2,
            'task_worker_num' => 4
        ]);
        
        $this->tcpServer->on('Receive', function ($serv, $fd, $reactorId, $data) use ($table) {
            // 异步处理数据
            $serv->task([
                'fd' => $fd,
                'data' => $data
            ]);
        });
        
        $this->tcpServer->on('Task', function ($serv, $taskId, $reactorId, $data) use ($table) {
            $metrics = json_decode($data['data'], true);
            
            foreach ($metrics as $metric => $value) {
                $table->set($metric, [
                    'value' => $value,
                    'timestamp' => time()
                ]);
            }
            
            // 通知WebSocket服务器有新数据
            $this->webSocketServer->push(1, json_encode(['update' => true]));
        });
        
        // WebSocket服务器配置
        $this->webSocketServer->on('Message', function ($server, $frame) use ($table) {
            $request = json_decode($frame->data, true);
            
            if ($request['type'] === 'get_metrics') {
                $metrics = [];
                foreach ($request['metrics'] as $metric) {
                    if ($table->exist($metric)) {
                        $metrics[$metric] = $table->get($metric);
                    }
                }
                
                $server->push($frame->fd, json_encode([
                    'type' => 'metrics_data',
                    'data' => $metrics
                ]));
            }
        });
    }
    
    public function start()
    {
        $this->tcpServer->start();
        $this->webSocketServer->start();
    }
}

$monitor = new MonitoringSystem();
$monitor->start();
```

## 3. 高并发API网关

### 架构设计

```
客户端请求 → API网关 → 负载均衡 → 微服务集群 → 响应返回
```

### 实现代码

```php
class APIGateway
{
    private $httpServer;
    private $services = [
        'user' => ['host' => '127.0.0.1', 'port' => 9504],
        'order' => ['host' => '127.0.0.1', 'port' => 9505]
    ];
    private $connectionPool = [];
    
    public function __construct()
    {
        $this->httpServer = new Swoole\Http\Server('0.0.0.0', 9501);
        $this->httpServer->set([
            'worker_num' => swoole_cpu_num() * 2,
            'enable_coroutine' => true,
            'http_compression' => true
        ]);
        
        $this->initConnectionPools();
        $this->setupRoutes();
    }
    
    private function initConnectionPools()
    {
        foreach ($this->services as $service => $config) {
            $this->connectionPool[$service] = new Co\Channel(10);
            
            go(function () use ($service, $config) {
                for ($i = 0; $i < 10; $i++) {
                    $client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
                    $client->connect($config['host'], $config['port']);
                    $this->connectionPool[$service]->push($client);
                }
            });
        }
    }
    
    private function setupRoutes()
    {
        $this->httpServer->on('Request', function ($request, $response) {
            $path = $request->server['request_uri'];
            $method = $request->server['request_method'];
            
            // 路由解析
            if (preg_match('/^\/user\/(.*)/', $path, $matches)) {
                $service = 'user';
                $action = $matches[1];
            } elseif (preg_match('/^\/order\/(.*)/', $path, $matches)) {
                $service = 'order';
                $action = $matches[1];
            } else {
                $response->status(404);
                $response->end('Not Found');
                return;
            }
            
            // 从连接池获取客户端
            $client = $this->connectionPool[$service]->pop();
            
            // 转发请求
            $client->send(json_encode([
                'action' => $action,
                'method' => $method,
                'params' => $request->get ?? [],
                'data' => $request->post ?? []
            ]));
            
            // 获取响应
            $result = $client->recv();
            
            // 归还连接
            $this->connectionPool[$service]->push($client);
            
            // 返回响应
            $response->header('Content-Type', 'application/json');
            $response->end($result);
        });
    }
    
    public function start()
    {
        $this->httpServer->start();
    }
}

$gateway = new APIGateway();
$gateway->start();
```

## 4. 实时日志收集系统

### 架构设计

```
应用日志 → UDP发送 → Swoole日志收集器 → Kafka队列 → 日志处理集群 → 存储/分析
```

### 实现代码

```php
class LogCollector
{
    private $server;
    private $kafkaProducer;
    
    public function __construct()
    {
        $this->server = new Swoole\Server('0.0.0.0', 9506, SWOOLE_PROCESS, SWOOLE_SOCK_UDP);
        $this->server->set([
            'worker_num' => 4,
            'enable_coroutine' => true
        ]);
        
        $this->kafkaProducer = new RdKafka\Producer();
        $this->kafkaProducer->addBrokers('kafka:9092');
        
        $this->setupServer();
    }
    
    private function setupServer()
    {
        $this->server->on('Packet', function ($server, $data, $clientInfo) {
            $logData = json_decode($data, true);
            
            // 验证日志格式
            if (!isset($logData['app'], $logData['level'], $logData['message'])) {
                return;
            }
            
            // 添加元数据
            $logData['timestamp'] = time();
            $logData['source_ip'] = $clientInfo['address'];
            
            // 根据日志级别发送到不同Kafka主题
            $topic = $this->kafkaProducer->newTopic($logData['app'] . '_' . $logData['level']);
            $topic->produce(RD_KAFKA_PARTITION_UA, 0, json_encode($logData));
            
            // 统计日志量
            $server->stats['log_count'] = ($server->stats['log_count'] ?? 0) + 1;
        });
        
        // 定时输出统计信息
        Swoole\Timer::tick(5000, function () use ($server) {
            echo "Logs processed: " . ($server->stats['log_count'] ?? 0) . "\n";
            $server->stats['log_count'] = 0;
        });
    }
    
    public function start()
    {
        $this->server->start();
    }
}

$collector = new LogCollector();
$collector->start();
```

## 5. 协程化爬虫系统

### 架构设计

```
调度器 → URL队列 → 协程爬虫Worker → 解析内容 → 存储结果
```

### 实现代码

```php
class WebCrawler
{
    private $concurrency = 10;
    private $queue;
    private $visited;
    
    public function __construct()
    {
        $this->queue = new Co\Channel(1000);
        $this->visited = new Swoole\Table(10000);
        $this->visited->column('url', Swoole\Table::TYPE_STRING, 256);
        $this->visited->create();
    }
    
    public function crawl($startUrl, $depth = 3)
    {
        $this->queue->push(['url' => $startUrl, 'depth' => 0]);
        
        Co\run(function () {
            for ($i = 0; $i < $this->concurrency; $i++) {
                go(function () {
                    while (true) {
                        $task = $this->queue->pop();
                        if ($task === false) break;
                        
                        $this->processUrl($task['url'], $task['depth']);
                    }
                });
            }
        });
    }
    
    private function processUrl($url, $depth)
    {
        if ($this->visited->exist(md5($url)) || $depth > 3) {
            return;
        }
        
        $this->visited->set(md5($url), ['url' => $url]);
        
        $httpClient = new Swoole\Coroutine\Http\Client(
            parse_url($url, PHP_URL_HOST),
            parse_url($url, PHP_URL_PORT) ?? 80
        );
        
        $httpClient->get(parse_url($url, PHP_URL_PATH));
        
        if ($httpClient->statusCode === 200) {
            $this->saveContent($url, $httpClient->body);
            
            // 解析页面中的链接
            $links = $this->extractLinks($httpClient->body);
            foreach ($links as $link) {
                $absoluteUrl = $this->makeAbsoluteUrl($url, $link);
                $this->queue->push([
                    'url' => $absoluteUrl,
                    'depth' => $depth + 1
                ]);
            }
        }
        
        $httpClient->close();
    }
    
    private function extractLinks($html)
    {
        $links = [];
        if (preg_match_all('/<a\s+.*?href=["\'](.*?)["\']/i', $html, $matches)) {
            $links = $matches[1];
        }
        return array_filter($links);
    }
    
    private function makeAbsoluteUrl($baseUrl, $relativeUrl)
    {
        // URL标准化处理
        return $relativeUrl; // 简化实现
    }
    
    private function saveContent($url, $content)
    {
        // 存储到数据库或文件
        echo "Crawled: {$url} (Size: " . strlen($content) . ")\n";
    }
}

$crawler = new WebCrawler();
$crawler->crawl('https://example.com');
```


