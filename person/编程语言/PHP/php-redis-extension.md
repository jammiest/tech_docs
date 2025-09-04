# PHP Redis 扩展详解

Redis 是一个高性能的键值存储系统，PHP 通过 Redis 扩展可以与 Redis 服务器进行交互。以下是 PHP Redis 扩展的全面介绍。

## 安装 Redis 扩展

### Linux 安装

```bash
# 安装依赖
sudo apt-get install php-dev

# 下载并安装 Redis 扩展
pecl install redis

# 在 php.ini 中添加扩展
echo "extension=redis.so" | sudo tee -a /etc/php/7.x/cli/php.ini
```

### Windows 安装

1. 下载对应版本的 php_redis.dll
2. 放入 PHP 的 ext 目录
3. 在 php.ini 中添加 `extension=php_redis.dll`

## 基本使用

### 连接 Redis 服务器

```php
<?php
$redis = new Redis();

// 简单连接
$redis->connect('127.0.0.1', 6379);

// 带超时和认证的连接
$redis->connect('127.0.0.1', 6379, 2.5); // 2.5秒超时
$redis->auth('password'); // 认证

// 检查连接
if (!$redis->ping()) {
    die("无法连接到Redis服务器");
}
```

## 基本操作

### 字符串操作

```php
// 设置键值
$redis->set('username', 'john_doe');

// 获取值
echo $redis->get('username'); // 输出: john_doe

// 设置带过期时间（秒）
$redis->setex('temp_key', 3600, 'temporary_value');

// 递增
$redis->incr('counter');
$redis->incrBy('counter', 5);

// 检查键是否存在
if ($redis->exists('username')) {
    echo "键存在";
}
```

### 哈希操作

```php
// 设置哈希字段
$redis->hSet('user:1000', 'name', 'John');
$redis->hSet('user:1000', 'email', 'john@example.com');

// 获取哈希字段
echo $redis->hGet('user:1000', 'name'); // 输出: John

// 获取所有字段
$userData = $redis->hGetAll('user:1000');
print_r($userData);
```

### 列表操作

```php
// 添加元素到列表
$redis->rPush('messages', 'msg1');
$redis->rPush('messages', 'msg2');

// 获取列表长度
echo $redis->lLen('messages'); // 输出: 2

// 获取列表元素
$messages = $redis->lRange('messages', 0, -1);
print_r($messages);
```

### 集合操作

```php
// 添加元素到集合
$redis->sAdd('tags', 'php');
$redis->sAdd('tags', 'redis');
$redis->sAdd('tags', 'database');

// 检查元素是否存在
if ($redis->sIsMember('tags', 'php')) {
    echo "php 在标签集合中";
}

// 获取所有元素
$tags = $redis->sMembers('tags');
print_r($tags);
```

### 有序集合操作

```php
// 添加带分数的元素
$redis->zAdd('rankings', 100, 'player1');
$redis->zAdd('rankings', 200, 'player2');

// 获取排名
$rankings = $redis->zRange('rankings', 0, -1, true);
print_r($rankings);
```

## 高级特性

### 发布/订阅

```php
// 订阅
$redis->subscribe(['channel1'], function ($redis, $channel, $message) {
    echo "收到来自 {$channel} 的消息: {$message}\n";
});

// 发布
$redis->publish('channel1', 'Hello, subscribers!');
```

### 事务

```php
// 开始事务
$redis->multi();

// 执行多个操作
$redis->set('key1', 'value1');
$redis->incr('counter');
$redis->get('key1');

// 执行事务
$result = $redis->exec();
print_r($result);
```

### Lua 脚本

```php
$script = "
    local key = KEYS[1]
    local value = ARGV[1]
    return redis.call('set', key, value)
";

$result = $redis->eval($script, ['mykey', 'myvalue'], 1);
```

## 连接池和持久连接

```php
// 持久连接
$redis->pconnect('127.0.0.1', 6379);

// 设置连接超时（毫秒）
$redis->setOption(Redis::OPT_READ_TIMEOUT, 1000);
```

## 错误处理

```php
try {
    $redis->connect('127.0.0.1', 6379);
    $redis->set('key', 'value');
} catch (RedisException $e) {
    echo "Redis错误: " . $e->getMessage();
    error_log($e->getMessage());
}
```

## 性能优化建议

1. **使用管道**减少网络往返
   ```php
   $redis->pipeline()
       ->set('key1', 'value1')
       ->set('key2', 'value2')
       ->exec();
   ```

2. **批量操作**代替多个单独操作

3. **合理设置过期时间**避免内存浪费

4. **使用合适的数据结构**根据场景选择

5. **避免大键**拆分大数据

## Redis 常用方法参考

| 方法 | 描述 |
|------|------|
| `set()` | 设置字符串键值 |
| `get()` | 获取字符串值 |
| `hSet()` | 设置哈希字段 |
| `hGet()` | 获取哈希字段 |
| `rPush()` | 向列表尾部添加元素 |
| `lRange()` | 获取列表元素 |
| `sAdd()` | 向集合添加元素 |
| `zAdd()` | 向有序集合添加元素 |
| `expire()` | 设置键过期时间 |
| `del()` | 删除键 |
| `multi()` | 开始事务 |
| `exec()` | 执行事务 |

PHP Redis 扩展提供了全面的 Redis 功能支持，是构建高性能应用的理想选择。