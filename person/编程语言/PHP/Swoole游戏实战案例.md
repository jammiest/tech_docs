# Swoole 游戏服务器实战案例

下面将介绍几个基于 Swoole 的游戏服务器开发经典案例，涵盖不同类型的游戏架构设计。

## 1. 多人在线游戏大厅系统

### 架构设计

```
客户端 ↔ WebSocket连接 ↔ 游戏大厅服务器 ↔ Redis(房间/玩家状态) ↔ MySQL(持久化数据)
```

### 实现代码

```php
class GameLobbyServer
{
    private $server;
    private $redis;
    private $rooms = [];
    
    public function __construct()
    {
        $this->server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
        $this->server->set([
            'worker_num' => swoole_cpu_num(),
            'enable_coroutine' => true,
            'heartbeat_idle_time' => 300,
            'heartbeat_check_interval' => 60
        ]);
        
        $this->redis = new Swoole\Coroutine\Redis();
        $this->redis->connect('127.0.0.1', 6379);
        
        $this->setupEvents();
    }
    
    private function setupEvents()
    {
        $this->server->on('Open', function ($server, $request) {
            $playerId = $request->get['player_id'];
            $this->redis->hSet('online_players', $request->fd, $playerId);
            echo "玩家 {$playerId} 进入大厅 (FD: {$request->fd})\n";
            
            // 发送大厅状态
            $server->push($request->fd, json_encode([
                'type' => 'lobby_status',
                'rooms' => $this->getRoomList()
            ]));
        });
        
        $this->server->on('Message', function ($server, $frame) {
            $data = json_decode($frame->data, true);
            
            switch ($data['action']) {
                case 'create_room':
                    $this->createRoom($server, $frame->fd, $data);
                    break;
                    
                case 'join_room':
                    $this->joinRoom($server, $frame->fd, $data);
                    break;
                    
                case 'leave_room':
                    $this->leaveRoom($server, $frame->fd);
                    break;
                    
                case 'room_chat':
                    $this->broadcastRoomMessage($server, $frame->fd, $data['message']);
                    break;
            }
        });
        
        $this->server->on('Close', function ($server, $fd) {
            $this->leaveRoom($server, $fd);
            $playerId = $this->redis->hGet('online_players', $fd);
            $this->redis->hDel('online_players', $fd);
            echo "玩家 {$playerId} 离开大厅 (FD: {$fd})\n";
        });
    }
    
    private function createRoom($server, $fd, $data)
    {
        $roomId = uniqid();
        $playerId = $this->redis->hGet('online_players', $fd);
        
        $this->rooms[$roomId] = [
            'id' => $roomId,
            'name' => $data['room_name'],
            'owner' => $playerId,
            'players' => [$fd],
            'game_type' => $data['game_type'],
            'max_players' => $data['max_players']
        ];
        
        $server->push($fd, json_encode([
            'type' => 'room_created',
            'room' => $this->rooms[$roomId]
        ]));
        
        $this->broadcastLobbyUpdate($server);
    }
    
    private function joinRoom($server, $fd, $data)
    {
        $roomId = $data['room_id'];
        $playerId = $this->redis->hGet('online_players', $fd);
        
        if (!isset($this->rooms[$roomId])) {
            $server->push($fd, json_encode([
                'type' => 'error',
                'message' => '房间不存在'
            ]));
            return;
        }
        
        if (count($this->rooms[$roomId]['players']) >= $this->rooms[$roomId]['max_players']) {
            $server->push($fd, json_encode([
                'type' => 'error',
                'message' => '房间已满'
            ]));
            return;
        }
        
        $this->rooms[$roomId]['players'][] = $fd;
        
        // 通知房间内所有玩家
        $this->broadcastRoomUpdate($server, $roomId);
        
        // 通知大厅其他玩家
        $this->broadcastLobbyUpdate($server);
    }
    
    private function broadcastRoomUpdate($server, $roomId)
    {
        $room = $this->rooms[$roomId];
        $players = [];
        
        foreach ($room['players'] as $fd) {
            $players[] = $this->redis->hGet('online_players', $fd);
        }
        
        $room['players'] = $players;
        
        foreach ($this->rooms[$roomId]['players'] as $fd) {
            if ($server->exist($fd)) {
                $server->push($fd, json_encode([
                    'type' => 'room_update',
                    'room' => $room
                ]));
            }
        }
    }
    
    public function start()
    {
        $this->server->start();
    }
}

$lobby = new GameLobbyServer();
$lobby->start();
```

## 2. 实时对战游戏服务器（类IO游戏）

### 架构设计

```
客户端 ↔ WebSocket ↔ 游戏房间服务器 ↔ UDP广播 ↔ 物理引擎 ↔ 状态同步
```

### 实现代码

```php
class BattleGameServer
{
    private $server;
    private $rooms = [];
    private $physicsEngine;
    
    public function __construct()
    {
        $this->server = new Swoole\WebSocket\Server("0.0.0.0", 9502);
        $this->server->set([
            'worker_num' => swoole_cpu_num(),
            'enable_coroutine' => true
        ]);
        
        $this->physicsEngine = new PhysicsEngine();
        
        $this->setupEvents();
    }
    
    private function setupEvents()
    {
        $this->server->on('Open', function ($server, $request) {
            $playerId = $request->get['player_id'];
            $roomId = $request->get['room_id'];
            
            if (!isset($this->rooms[$roomId])) {
                $this->rooms[$roomId] = [
                    'players' => [],
                    'game_state' => 'waiting',
                    'physics_world' => $this->physicsEngine->createWorld()
                ];
            }
            
            $this->rooms[$roomId]['players'][$request->fd] = [
                'id' => $playerId,
                'position' => ['x' => 0, 'y' => 0],
                'velocity' => ['x' => 0, 'y' => 0],
                'inputs' => []
            ];
            
            // 启动游戏循环（如果还没启动）
            if ($this->rooms[$roomId]['game_state'] === 'waiting' && 
                count($this->rooms[$roomId]['players']) >= 2) {
                $this->startGameLoop($server, $roomId);
            }
        });
        
        $this->server->on('Message', function ($server, $frame) {
            $data = json_decode($frame->data, true);
            $playerId = $data['player_id'];
            $roomId = $data['room_id'];
            
            if (isset($this->rooms[$roomId]['players'][$frame->fd])) {
                // 记录玩家输入
                $this->rooms[$roomId]['players'][$frame->fd]['inputs'] = $data['inputs'];
            }
        });
        
        $this->server->on('Close', function ($server, $fd) {
            foreach ($this->rooms as $roomId => $room) {
                if (isset($room['players'][$fd])) {
                    unset($this->rooms[$roomId]['players'][$fd]);
                    
                    // 通知其他玩家
                    $this->broadcastRoomState($server, $roomId, [
                        'type' => 'player_left',
                        'player_id' => $room['players'][$fd]['id']
                    ]);
                    
                    // 如果房间空了，清理资源
                    if (empty($this->rooms[$roomId]['players'])) {
                        $this->physicsEngine->destroyWorld($this->rooms[$roomId]['physics_world']);
                        unset($this->rooms[$roomId]);
                    }
                    
                    break;
                }
            }
        });
    }
    
    private function startGameLoop($server, $roomId)
    {
        $this->rooms[$roomId]['game_state'] = 'running';
        
        go(function () use ($server, $roomId) {
            $lastUpdate = microtime(true);
            $frameRate = 60; // 60FPS
            
            while (isset($this->rooms[$roomId]) && 
                   $this->rooms[$roomId]['game_state'] === 'running') {
                $start = microtime(true);
                
                // 物理引擎更新
                $this->physicsEngine->update(
                    $this->rooms[$roomId]['physics_world'],
                    1 / $frameRate
                );
                
                // 处理玩家输入
                foreach ($this->rooms[$roomId]['players'] as $fd => &$player) {
                    $this->applyPlayerInputs($player);
                    
                    // 同步状态给客户端
                    if ($server->exist($fd)) {
                        $server->push($fd, json_encode([
                            'type' => 'game_update',
                            'players' => $this->getPlayersState($roomId),
                            'world' => $this->rooms[$roomId]['physics_world']
                        ]));
                    }
                }
                
                // 控制帧率
                $elapsed = microtime(true) - $start;
                $sleepTime = max(0, (1000 / $frameRate) - ($elapsed * 1000));
                if ($sleepTime > 0) {
                    Co::sleep($sleepTime / 1000);
                }
            }
        });
    }
    
    private function applyPlayerInputs(&$player)
    {
        // 根据输入更新玩家状态
        if (isset($player['inputs']['up'])) {
            $player['velocity']['y'] = -5;
        }
        if (isset($player['inputs']['down'])) {
            $player['velocity']['y'] = 5;
        }
        if (isset($player['inputs']['left'])) {
            $player['velocity']['x'] = -5;
        }
        if (isset($player['inputs']['right'])) {
            $player['velocity']['x'] = 5;
        }
        
        // 应用速度到位置
        $player['position']['x'] += $player['velocity']['x'];
        $player['position']['y'] += $player['velocity']['y'];
        
        // 重置速度
        $player['velocity'] = ['x' => 0, 'y' => 0];
    }
    
    public function start()
    {
        $this->server->start();
    }
}

$battleServer = new BattleGameServer();
$battleServer->start();
```

## 3. 卡牌游戏服务器（回合制）

### 架构设计

```
客户端 ↔ WebSocket ↔ 游戏房间服务器 ↔ 状态机引擎 ↔ 事件系统 ↔ 数据库
```

### 实现代码

```php
class CardGameServer
{
    private $server;
    private $rooms = [];
    private $gameLogic;
    
    public function __construct()
    {
        $this->server = new Swoole\WebSocket\Server("0.0.0.0", 9503);
        $this->server->set([
            'worker_num' => swoole_cpu_num(),
            'enable_coroutine' => true
        ]);
        
        $this->gameLogic = new CardGameLogic();
        
        $this->setupEvents();
    }
    
    private function setupEvents()
    {
        $this->server->on('Open', function ($server, $request) {
            $playerId = $request->get['player_id'];
            $roomId = $request->get['room_id'];
            
            if (!isset($this->rooms[$roomId])) {
                $this->rooms[$roomId] = [
                    'players' => [],
                    'game_state' => $this->gameLogic->newGameState(),
                    'timer' => null
                ];
            }
            
            $this->rooms[$roomId]['players'][$request->fd] = [
                'id' => $playerId,
                'ready' => false
            ];
            
            // 如果房间已满，开始游戏
            if (count($this->rooms[$roomId]['players']) >= 2 && 
                $this->rooms[$roomId]['game_state']['status'] === 'waiting') {
                $this->startGame($server, $roomId);
            }
        });
        
        $this->server->on('Message', function ($server, $frame) {
            $data = json_decode($frame->data, true);
            $playerId = $data['player_id'];
            $roomId = $data['room_id'];
            
            if (!isset($this->rooms[$roomId]['players'][$frame->fd])) {
                return;
            }
            
            switch ($data['action']) {
                case 'ready':
                    $this->rooms[$roomId]['players'][$frame->fd]['ready'] = true;
                    $this->broadcastRoomState($server, $roomId);
                    break;
                    
                case 'play_card':
                    $this->handlePlayCard($server, $roomId, $frame->fd, $data);
                    break;
                    
                case 'end_turn':
                    $this->handleEndTurn($server, $roomId, $frame->fd);
                    break;
            }
        });
        
        $this->server->on('Close', function ($server, $fd) {
            foreach ($this->rooms as $roomId => $room) {
                if (isset($room['players'][$fd])) {
                    // 结束游戏或标记玩家离线
                    $this->rooms[$roomId]['game_state']['status'] = 'paused';
                    
                    // 通知其他玩家
                    $this->broadcastRoomState($server, $roomId, [
                        'type' => 'player_disconnected',
                        'player_id' => $room['players'][$fd]['id']
                    ]);
                    
                    // 清理定时器
                    if ($this->rooms[$roomId]['timer']) {
                        Swoole\Timer::clear($this->rooms[$roomId]['timer']);
                    }
                    
                    break;
                }
            }
        });
    }
    
    private function startGame($server, $roomId)
    {
        $playerIds = [];
        foreach ($this->rooms[$roomId]['players'] as $fd => $player) {
            $playerIds[] = $player['id'];
        }
        
        $this->rooms[$roomId]['game_state'] = $this->gameLogic->startGame($playerIds);
        $this->broadcastRoomState($server, $roomId);
        
        // 设置回合定时器
        $this->rooms[$roomId]['timer'] = Swoole\Timer::tick(30000, function () use ($server, $roomId) {
            if ($this->rooms[$roomId]['game_state']['status'] === 'playing') {
                $this->gameLogic->timeout($this->rooms[$roomId]['game_state']);
                $this->broadcastRoomState($server, $roomId);
            }
        });
    }
    
    private function handlePlayCard($server, $roomId, $fd, $data)
    {
        $playerId = $this->rooms[$roomId]['players'][$fd]['id'];
        
        try {
            $this->gameLogic->playCard(
                $this->rooms[$roomId]['game_state'],
                $playerId,
                $data['card_id'],
                $data['target'] ?? null
            );
            
            $this->broadcastRoomState($server, $roomId);
            
            // 检查游戏是否结束
            if ($this->rooms[$roomId]['game_state']['status'] === 'finished') {
                $this->endGame($server, $roomId);
            }
        } catch (GameException $e) {
            $server->push($fd, json_encode([
                'type' => 'error',
                'message' => $e->getMessage()
            ]));
        }
    }
    
    private function endGame($server, $roomId)
    {
        // 保存游戏结果
        $this->saveGameResult($this->rooms[$roomId]['game_state']);
        
        // 通知所有玩家
        $this->broadcastRoomState($server, $roomId);
        
        // 清理定时器
        if ($this->rooms[$roomId]['timer']) {
            Swoole\Timer::clear($this->rooms[$roomId]['timer']);
        }
        
        // 10秒后关闭房间
        Swoole\Timer::after(10000, function () use ($roomId) {
            unset($this->rooms[$roomId]);
        });
    }
    
    private function broadcastRoomState($server, $roomId, $additionalData = [])
    {
        $state = $this->rooms[$roomId]['game_state'];
        $players = $this->rooms[$roomId]['players'];
        
        foreach ($players as $fd => $player) {
            if ($server->exist($fd)) {
                $playerState = $this->gameLogic->filterStateForPlayer($state, $player['id']);
                
                $server->push($fd, json_encode(array_merge([
                    'type' => 'game_state',
                    'state' => $playerState,
                    'players' => array_column($players, 'id')
                ], $additionalData)));
            }
        }
    }
    
    public function start()
    {
        $this->server->start();
    }
}

$cardServer = new CardGameServer();
$cardServer->start();
```

## 4. MMORPG 游戏网关服务器

### 架构设计

```
客户端 ↔ WebSocket ↔ 网关服务器 ↔ TCP ↔ 区域服务器 ↔ Redis(共享状态) ↔ MySQL
```

### 实现代码

```php
class MMORPGGateway
{
    private $server;
    private $zoneServers = [
        'zone1' => ['host' => '127.0.0.1', 'port' => 9504],
        'zone2' => ['host' => '127.0.0.1', 'port' => 9505]
    ];
    private $playerZones = [];
    
    public function __construct()
    {
        $this->server = new Swoole\WebSocket\Server("0.0.0.0", 9504);
        $this->server->set([
            'worker_num' => swoole_cpu_num(),
            'enable_coroutine' => true
        ]);
        
        $this->setupEvents();
    }
    
    private function setupEvents()
    {
        $this->server->on('Open', function ($server, $request) {
            $playerId = $request->get['player_id'];
            echo "玩家 {$playerId} 连接网关\n";
        });
        
        $this->server->on('Message', function ($server, $frame) {
            $data = json_decode($frame->data, true);
            $playerId = $data['player_id'];
            
            if (!isset($this->playerZones[$playerId])) {
                // 分配区域服务器
                $zoneId = $this->assignZoneServer($playerId, $data['position']);
                $this->playerZones[$playerId] = $zoneId;
            }
            
            // 转发消息到区域服务器
            $this->forwardToZoneServer(
                $this->playerZones[$playerId],
                $frame->fd,
                $data
            );
        });
        
        $this->server->on('Close', function ($server, $fd) {
            // 通知区域服务器玩家断开连接
            foreach ($this->playerZones as $playerId => $zoneId) {
                $this->forwardToZoneServer($zoneId, $fd, [
                    'action' => 'disconnect',
                    'player_id' => $playerId
                ]);
            }
        });
    }
    
    private function assignZoneServer($playerId, $position)
    {
        // 根据玩家位置分配区域服务器
        // 这里简化为轮询分配
        static $lastZone = 0;
        $zoneIds = array_keys($this->zoneServers);
        $zoneId = $zoneIds[$lastZone % count($zoneIds)];
        $lastZone++;
        
        echo "分配玩家 {$playerId} 到区域服务器 {$zoneId}\n";
        return $zoneId;
    }
    
    private function forwardToZoneServer($zoneId, $fd, $data)
    {
        go(function () use ($zoneId, $fd, $data) {
            $client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
            $client->connect(
                $this->zoneServers[$zoneId]['host'],
                $this->zoneServers[$zoneId]['port']
            );
            
            $data['gateway_fd'] = $fd;
            $client->send(json_encode($data));
            
            $response = $client->recv();
            $client->close();
            
            // 将区域服务器的响应返回给客户端
            if ($this->server->exist($fd)) {
                $this->server->push($fd, $response);
            }
        });
    }
    
    public function start()
    {
        $this->server->start();
    }
}

$gateway = new MMORPGGateway();
$gateway->start();
```

这些案例展示了 Swoole 在不同类型游戏服务器开发中的应用，从大厅系统到实时对战，从回合制卡牌到大型多人在线游戏。关键点包括：

1. **WebSocket 实时通信**：所有案例都基于 WebSocket 实现低延迟通信
2. **状态管理**：使用 Swoole\Table 或 Redis 管理游戏状态
3. **协程并发**：利用协程处理高并发连接
4. **分布式架构**：MMORPG 案例展示了网关与区域服务器的分离
5. **性能优化**：通过合理的进程/协程配置实现高性能

实际开发中还需要考虑：
- 安全验证机制
- 断线重连处理
- 反作弊系统
- 日志和监控
- 压力测试和性能优化