# Swoole 终极用法：构建企业级高性能服务

Swoole 的终极用法是将各种高级特性组合运用，构建高性能、高可用的企业级服务。下面将介绍几种终极应用场景和实现方案。

## 1. 微服务架构实现

### 服务注册与发现

```php
// 服务注册中心
$registry = new Swoole\Server('0.0.0.0', 9500);
$registry->set([
    'enable_coroutine' => true,
    'heartbeat_check_interval' => 60,
    'heartbeat_idle_time' => 120
]);

$services = new Swoole\Table(1024);
$services->column('host', Swoole\Table::TYPE_STRING, 32);
$services->column('port', Swoole\Table::TYPE_INT);
$services->column('last_heartbeat', Swoole\Table::TYPE_INT);
$services->create();

$registry->on('Receive', function ($serv, $fd, $reactorId, $data) use ($services) {
    $data = json_decode($data, true);
    switch ($data['cmd']) {
        case 'register':
            $services->set($data['service_name'], [
                'host' => $data['host'],
                'port' => $data['port'],
                'last_heartbeat' => time()
            ]);
            $serv->send($fd, json_encode(['code' => 0]));
            break;
        case 'heartbeat':
            $services->set($data['service_name'], [
                'last_heartbeat' => time()
            ]);
            break;
        case 'discover':
            $service = $services->get($data['service_name']);
            $serv->send($fd, json_encode($service));
            break;
    }
});
```

### 服务间通信

```php
class RPCClient
{
    private $pool;
    
    public function __construct($serviceName) {
        $this->pool = new Co\Channel(10);
        
        go(function () use ($serviceName) {
            $registry = new Co\Client(SWOOLE_SOCK_TCP);
            $registry->connect('127.0.0.1', 9500);
            $registry->send(json_encode([
                'cmd' => 'discover',
                'service_name' => $serviceName
            ]));
            $serviceInfo = json_decode($registry->recv(), true);
            
            for ($i = 0; $i < 10; $i++) {
                $client = new Co\Client(SWOOLE_SOCK_TCP);
                $client->connect($serviceInfo['host'], $serviceInfo['port']);
                $this->pool->push($client);
            }
        });
    }
    
    public function call($method, $params) {
        $client = $this->pool->pop();
        $client->send(json_encode([
            'method' => $method,
            'params' => $params
        ]));
        $result = json_decode($client->recv(), true);
        $this->pool->push($client);
        return $result;
    }
}
```

## 2. 高性能API网关

```php
class APIGateway
{
    private $server;
    private $routeTable;
    
    public function __construct() {
        $this->server = new Swoole\Http\Server('0.0.0.0', 9501);
        $this->server->set([
            'enable_coroutine' => true,
            'http_compression' => true,
            'buffer_output_size' => 32 * 1024 * 1024
        ]);
        
        $this->routeTable = new Swoole\Table(1024);
        $this->routeTable->column('service', Swoole\Table::TYPE_STRING, 32);
        $this->routeTable->column('method', Swoole\Table::TYPE_STRING, 16);
        $this->routeTable->create();
        
        $this->initRoutes();
    }
    
    private function initRoutes() {
        $routes = [
            '/user/login' => ['service' => 'user', 'method' => 'POST'],
            '/product/list' => ['service' => 'product', 'method' => 'GET']
        ];
        
        foreach ($routes as $path => $config) {
            $this->routeTable->set($path, $config);
        }
    }
    
    public function start() {
        $this->server->on('Request', function ($request, $response) {
            $path = $request->server['request_uri'];
            $route = $this->routeTable->get($path);
            
            if (!$route) {
                $response->status(404);
                $response->end('Not Found');
                return;
            }
            
            $result = Co\run(function () use ($route, $request) {
                $rpc = new RPCClient($route['service']);
                return $rpc->call($route['method'], $request->get);
            });
            
            $response->header('Content-Type', 'application/json');
            $response->end(json_encode($result));
        });
        
        $this->server->start();
    }
}
```

## 3. 实时大数据处理

### 流式数据处理管道

```php
class DataPipeline
{
    private $workers = [];
    private $channel;
    
    public function __construct($workerCount = 4) {
        $this->channel = new Co\Channel(10000);
        
        for ($i = 0; $i < $workerCount; $i++) {
            $this->workers[] = new Swoole\Process(function () {
                Co\run(function () {
                    while (true) {
                        $data = $this->channel->pop();
                        if ($data === false) break;
                        
                        // 数据处理逻辑
                        $result = $this->processData($data);
                        
                        // 写入结果
                        $this->writeResult($result);
                    }
                });
            });
        }
    }
    
    public function start() {
        foreach ($this->workers as $worker) {
            $worker->start();
        }
        
        // 数据源监听
        $server = new Swoole\Server('0.0.0.0', 9502);
        $server->on('Receive', function ($serv, $fd, $reactorId, $data) {
            $this->channel->push($data);
        });
        
        $server->start();
    }
    
    private function processData($data) {
        // 复杂的数据处理逻辑
        Co::sleep(0.1); // 模拟耗时操作
        return strtoupper($data);
    }
    
    private function writeResult($result) {
        // 写入数据库或文件
        file_put_contents('result.log', $result.PHP_EOL, FILE_APPEND);
    }
}
```

## 4. 游戏服务器架构

### 游戏房间管理

```php
class GameServer
{
    private $server;
    private $rooms;
    private $players;
    
    public function __construct() {
        $this->server = new Swoole\WebSocket\Server('0.0.0.0', 9503);
        $this->server->set([
            'enable_coroutine' => true,
            'heartbeat_idle_time' => 600,
            'heartbeat_check_interval' => 60
        ]);
        
        $this->rooms = new Swoole\Table(1024);
        $this->rooms->column('player_count', Swoole\Table::TYPE_INT);
        $this->rooms->column('status', Swoole\Table::TYPE_STRING, 16);
        $this->rooms->create();
        
        $this->players = new Swoole\Table(65536);
        $this->players->column('room_id', Swoole\Table::TYPE_INT);
        $this->players->column('status', Swoole\Table::TYPE_STRING, 16);
        $this->players->create();
        
        $this->initServerEvents();
    }
    
    private function initServerEvents() {
        $this->server->on('Open', function ($server, $request) {
            echo "Player {$request->fd} connected\n";
        });
        
        $this->server->on('Message', function ($server, $frame) {
            $data = json_decode($frame->data, true);
            switch ($data['action']) {
                case 'join_room':
                    $this->joinRoom($frame->fd, $data['room_id']);
                    break;
                case 'leave_room':
                    $this->leaveRoom($frame->fd);
                    break;
                case 'game_action':
                    $this->handleGameAction($frame->fd, $data);
                    break;
            }
        });
        
        $this->server->on('Close', function ($server, $fd) {
            $this->leaveRoom($fd);
            echo "Player {$fd} disconnected\n";
        });
    }
    
    private function joinRoom($fd, $roomId) {
        if (!$this->rooms->exist($roomId)) {
            $this->rooms->set($roomId, [
                'player_count' => 0,
                'status' => 'waiting'
            ]);
        }
        
        $this->players->set($fd, [
            'room_id' => $roomId,
            'status' => 'ready'
        ]);
        
        $this->rooms->incr($roomId, 'player_count');
        
        // 通知房间内所有玩家
        $this->broadcastRoom($roomId, [
            'action' => 'player_joined',
            'player_id' => $fd,
            'room_info' => $this->rooms->get($roomId)
        ]);
    }
    
    private function broadcastRoom($roomId, $message) {
        foreach ($this->players as $fd => $player) {
            if ($player['room_id'] == $roomId) {
                $this->server->push($fd, json_encode($message));
            }
        }
    }
    
    public function start() {
        $this->server->start();
    }
}
```

## 5. 金融级交易系统

### 高并发订单处理

```php
class TradingEngine
{
    private $server;
    private $orderBook;
    private $orderQueue;
    
    public function __construct() {
        $this->server = new Swoole\Server('0.0.0.0', 9504);
        $this->server->set([
            'enable_coroutine' => true,
            'reactor_num' => 8,
            'worker_num' => 16,
            'task_worker_num' => 8,
            'dispatch_mode' => 2
        ]);
        
        $this->orderBook = new Swoole\Table(1000000);
        $this->orderBook->column('symbol', Swoole\Table::TYPE_STRING, 16);
        $this->orderBook->column('price', Swoole\Table::TYPE_FLOAT);
        $this->orderBook->column('quantity', Swoole\Table::TYPE_INT);
        $this->orderBook->column('side', Swoole\Table::TYPE_STRING, 4);
        $this->orderBook->column('status', Swoole\Table::TYPE_STRING, 16);
        $this->orderBook->create();
        
        $this->orderQueue = new Co\Channel(100000);
        
        $this->initServerEvents();
    }
    
    private function initServerEvents() {
        $this->server->on('Receive', function ($server, $fd, $reactorId, $data) {
            $order = json_decode($data, true);
            $orderId = uniqid();
            
            $this->orderBook->set($orderId, [
                'symbol' => $order['symbol'],
                'price' => $order['price'],
                'quantity' => $order['quantity'],
                'side' => $order['side'],
                'status' => 'pending'
            ]);
            
            $this->orderQueue->push(['order_id' => $orderId, 'fd' => $fd]);
        });
        
        $this->server->on('Task', function ($server, $taskId, $reactorId, $data) {
            $orderId = $data['order_id'];
            $order = $this->orderBook->get($orderId);
            
            // 订单匹配逻辑
            $matched = $this->matchOrder($order);
            
            if ($matched) {
                $this->orderBook->set($orderId, ['status' => 'filled']);
            } else {
                $this->orderBook->set($orderId, ['status' => 'partial']);
            }
            
            $server->send($data['fd'], json_encode([
                'order_id' => $orderId,
                'status' => $this->orderBook->get($orderId)['status']
            ]));
        });
    }
    
    private function matchOrder($order) {
        // 复杂的订单匹配算法
        // 这里简化为随机匹配
        return rand(0, 1) == 1;
    }
    
    public function start() {
        // 启动订单处理协程
        go(function () {
            while (true) {
                $data = $this->orderQueue->pop();
                $this->server->task($data);
            }
        });
        
        $this->server->start();
    }
}
```

## 6. 物联网(IoT)平台

### 百万设备连接管理

```php
class IoTPlatform
{
    private $server;
    private $devices;
    private $deviceGroups;
    
    public function __construct() {
        $this->server = new Swoole\Server('0.0.0.0', 9505, SWOOLE_BASE, SWOOLE_SOCK_TCP);
        $this->server->set([
            'enable_coroutine' => true,
            'worker_num' => swoole_cpu_num() * 2,
            'max_conn' => 1000000,
            'buffer_output_size' => 16 * 1024 * 1024,
            'socket_buffer_size' => 128 * 1024 * 1024
        ]);
        
        $this->devices = new Swoole\Table(1000000);
        $this->devices->column('device_id', Swoole\Table::TYPE_STRING, 64);
        $this->devices->column('group_id', Swoole\Table::TYPE_STRING, 32);
        $this->devices->column('last_active', Swoole\Table::TYPE_INT);
        $this->devices->column('status', Swoole\Table::TYPE_STRING, 16);
        $this->devices->create();
        
        $this->deviceGroups = new Swoole\Table(1024);
        $this->deviceGroups->column('device_count', Swoole\Table::TYPE_INT);
        $this->deviceGroups->column('last_command', Swoole\Table::TYPE_STRING, 256);
        $this->deviceGroups->create();
        
        $this->initServerEvents();
    }
    
    private function initServerEvents() {
        $this->server->on('Connect', function ($server, $fd) {
            echo "Device {$fd} connected\n";
        });
        
        $this->server->on('Receive', function ($server, $fd, $reactorId, $data) {
            $message = json_decode($data, true);
            
            switch ($message['type']) {
                case 'register':
                    $this->registerDevice($fd, $message);
                    break;
                case 'heartbeat':
                    $this->updateDeviceStatus($fd, $message);
                    break;
                case 'data':
                    $this->processDeviceData($fd, $message);
                    break;
            }
        });
        
        $this->server->on('Close', function ($server, $fd) {
            $this->unregisterDevice($fd);
            echo "Device {$fd} disconnected\n";
        });
    }
    
    private function registerDevice($fd, $message) {
        $this->devices->set($fd, [
            'device_id' => $message['device_id'],
            'group_id' => $message['group_id'],
            'last_active' => time(),
            'status' => 'online'
        ]);
        
        if (!$this->deviceGroups->exist($message['group_id'])) {
            $this->deviceGroups->set($message['group_id'], [
                'device_count' => 0,
                'last_command' => ''
            ]);
        }
        
        $this->deviceGroups->incr($message['group_id'], 'device_count');
    }
    
    private function broadcastToGroup($groupId, $message) {
        foreach ($this->devices as $fd => $device) {
            if ($device['group_id'] == $groupId) {
                $this->server->send($fd, json_encode($message));
            }
        }
    }
    
    public function start() {
        // 启动设备状态检查协程
        go(function () {
            while (true) {
                Co::sleep(60);
                $now = time();
                foreach ($this->devices as $fd => $device) {
                    if ($now - $device['last_active'] > 300) {
                        $this->devices->set($fd, ['status' => 'offline']);
                    }
                }
            }
        });
        
        $this->server->start();
    }
}
```

这些终极用法展示了 Swoole 在企业级应用中的强大能力，通过合理组合各种高级特性，可以构建出高性能、高可用的复杂系统。