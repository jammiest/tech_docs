# ThinkPHP MySQL 调优方案

> 表结构在此：[电商系统核心表结构DDL](/person/数据库/MySQL/电商系统核心表结构DDL.md)


### 1. 主从复制与读写分离
```php
// config/database.php
return [
    'connections' => [
        'mysql' => [
            // 主库配置（写操作）
            'write' => [
                'host' => ['192.168.1.1'],
                'username' => 'write_user',
                'password' => 'write_password',
            ],
            // 从库配置（读操作）
            'read' => [
                'host' => ['192.168.1.2', '192.168.1.3'],
                'username' => 'read_user',
                'password' => 'read_password',
            ],
        ],
    ],
];
```

### 2. 分库分表策略
```php
// 订单表按用户ID哈希分表
class Order extends Model
{
    protected $connection = 'mysql';
    
    public function getTable()
    {
        $userId = $this->user_id ?? 0;
        $suffix = $userId % 10; // 分为10个表
        return 'orders_' . $suffix;
    }
}
```

## 二、索引优化方案

### 1. 核心表索引设计
```sql
-- 商品表索引
ALTER TABLE `goods` 
ADD INDEX `idx_category_sales` (`category_id`, `sales_count`),
ADD INDEX `idx_brand_status` (`brand_id`, `is_on_sale`),
ADD INDEX `idx_price_range` (`shop_price`, `market_price`);

-- 订单表索引
ALTER TABLE `orders`
ADD INDEX `idx_user_status` (`user_id`, `order_status`),
ADD INDEX `idx_create_time` (`created_at`),
ADD INDEX `idx_pay_time` (`pay_time`);

-- 订单商品表索引
ALTER TABLE `order_items`
ADD INDEX `idx_order_goods` (`order_id`, `goods_id`),
ADD INDEX `idx_sku_sales` (`sku_id`, `created_at`);
```

### 2. 复合索引设计原则
```sql
-- 遵循最左前缀原则
-- 商品搜索复合索引
ALTER TABLE `goods`
ADD INDEX `idx_search` (`category_id`, `is_on_sale`, `shop_price`, `sales_count`);

-- 订单查询复合索引
ALTER TABLE `orders`
ADD INDEX `idx_user_query` (`user_id`, `order_status`, `created_at`);
```

## 三、SQL 查询优化

### 1. 避免全表扫描
```php
// 错误示例（无索引字段查询）
$orders = Db::name('orders')->where('order_sn', 'like', '%2023%')->select();

// 优化方案
$orders = Db::name('orders')
    ->where('order_sn', 'like', '2023%') // 左前缀匹配
    ->select();
```

### 2. JOIN 优化
```php
// 错误示例（JOIN过多）
$list = Db::name('orders')
    ->alias('o')
    ->join('users u', 'o.user_id = u.id')
    ->join('user_addresses ua', 'o.address_id = ua.id')
    ->join('order_items oi', 'o.id = oi.order_id')
    ->join('goods g', 'oi.goods_id = g.id')
    ->select();

// 优化方案（分步查询+缓存）
$orders = Db::name('orders')->where('user_id', $userId)->select();
$orderIds = array_column($orders, 'id');
$items = Db::name('order_items')
    ->where('order_id', 'in', $orderIds)
    ->select();
```

### 3. 分页优化
```php
// 传统分页问题（大数据量OFFSET性能差）
$list = Db::name('orders')->page($page, $size)->select();

// 优化方案1：ID分页
$lastId = request()->param('last_id', 0);
$list = Db::name('orders')
    ->where('id', '>', $lastId)
    ->limit($size)
    ->order('id', 'asc')
    ->select();

// 优化方案2：延迟关联
$subQuery = Db::name('orders')
    ->field('id')
    ->where('user_id', $userId)
    ->order('id', 'desc')
    ->page($page, $size)
    ->buildSql();
    
$list = Db::table($subQuery . ' a')
    ->join('orders o', 'a.id = o.id')
    ->select();
```

## 四、数据库配置优化

### 1. my.cnf 关键配置
```ini
[mysqld]
# 基础配置
innodb_buffer_pool_size = 4G  # 总内存的50-70%
innodb_buffer_pool_instances = 8
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1

# 连接配置
max_connections = 500
thread_cache_size = 50
table_open_cache = 2000

# 查询缓存
query_cache_type = 0  # 电商系统建议关闭
query_cache_size = 0

# InnoDB配置
innodb_file_per_table = ON
innodb_flush_method = O_DIRECT
innodb_read_io_threads = 8
innodb_write_io_threads = 4
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000
```

### 2. ThinkPHP 数据库配置
```php
// config/database.php
return [
    // 数据库调试模式
    'debug' => false,
    
    // 数据库部署方式:0 集中式,1 分布式
    'deploy' => 1,
    
    // 开启断线重连
    'break_reconnect' => true,
    
    // 查询缓存
    'fields_cache' => false, // 生产环境建议关闭
    
    // PDO参数
    'params' => [
        PDO::ATTR_EMULATE_PREPARES => false, // 禁用预处理模拟
        PDO::ATTR_STRINGIFY_FETCHES => false,
    ],
];
```

## 五、慢查询分析与优化

### 1. 开启慢查询日志
```ini
# my.cnf
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 1
log_queries_not_using_indexes = 1
```

### 2. 使用 pt-query-digest 分析
```bash
pt-query-digest /var/log/mysql/mysql-slow.log
```

### 3. 常见慢查询优化案例
```sql
-- 案例1：大表分页优化
-- 原SQL
SELECT * FROM orders ORDER BY id DESC LIMIT 100000, 20;

-- 优化后
SELECT * FROM orders WHERE id < ? ORDER BY id DESC LIMIT 20;

-- 案例2：IN子查询优化
-- 原SQL
SELECT * FROM goods WHERE id IN (SELECT goods_id FROM order_items WHERE order_id = 100);

-- 优化后
SELECT g.* FROM goods g JOIN order_items oi ON g.id = oi.goods_id WHERE oi.order_id = 100;
```

## 六、事务与锁优化

### 1. 事务优化
```php
// 减少事务范围
Db::startTrans();
try {
    // 只包含必要的操作
    $order = Order::create($data);
    OrderItem::insertAll($items);
    Inventory::where('sku_id', $skuId)->dec('stock', $quantity);
    
    Db::commit();
} catch (\Exception $e) {
    Db::rollback();
    throw $e;
}
```

### 2. 死锁预防
```php
// 统一获取锁的顺序
public function createOrder()
{
    // 按照商品ID排序获取锁
    $skuIds = array_column($items, 'sku_id');
    sort($skuIds);
    
    foreach ($skuIds as $skuId) {
        $lock = Redis::lock('sku_' . $skuId, 10);
        if (!$lock->get()) {
            throw new Exception('获取库存锁失败');
        }
    }
    
    // 业务逻辑...
}
```

## 七、数据归档与分区

### 1. 历史数据归档
```sql
-- 创建归档表
CREATE TABLE orders_archive LIKE orders;

-- 归档数据
INSERT INTO orders_archive 
SELECT * FROM orders 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- 删除已归档数据
DELETE FROM orders 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

### 2. 表分区方案
```sql
-- 订单表按时间范围分区
ALTER TABLE orders PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);
```

## 八、缓存与数据库协同

### 1. 多级缓存策略
```php
// 商品详情获取逻辑
public function getGoodsDetail($id)
{
    // 1. 检查本地缓存
    $cacheKey = 'goods_detail_' . $id;
    $data = Cache::get($cacheKey);
    
    if ($data) {
        return $data;
    }
    
    // 2. 检查Redis缓存
    $data = Redis::get($cacheKey);
    if ($data) {
        Cache::set($cacheKey, $data, 60); // 本地缓存1分钟
        return $data;
    }
    
    // 3. 查询数据库
    $data = Db::name('goods')
        ->with(['skus', 'brand', 'category'])
        ->find($id);
    
    // 4. 写入缓存
    Redis::setex($cacheKey, 3600, $data); // Redis缓存1小时
    Cache::set($cacheKey, $data, 60); // 本地缓存1分钟
    
    return $data;
}
```

### 2. 库存缓存方案
```php
// 库存检查与扣减
public function decreaseStock($skuId, $quantity)
{
    $lockKey = 'stock_lock_' . $skuId;
    $cacheKey = 'stock_' . $skuId;
    
    // 获取分布式锁
    $lock = Redis::set($lockKey, 1, ['nx', 'ex' => 5]);
    if (!$lock) {
        throw new Exception('系统繁忙，请重试');
    }
    
    try {
        // 使用Lua脚本保证原子性
        $script = "
            local stock = tonumber(redis.call('GET', KEYS[1]))
            local quantity = tonumber(ARGV[1])
            
            if stock and stock >= quantity then
                redis.call('DECRBY', KEYS[1], quantity)
                return 1
            else
                return 0
            end
        ";
        
        $result = Redis::eval($script, 1, $cacheKey, $quantity);
        
        if ($result === 0) {
            throw new Exception('库存不足');
        }
        
        // 异步更新数据库
        Queue::push(new UpdateStockJob($skuId, -$quantity));
        
        return true;
    } finally {
        Redis::del($lockKey);
    }
}
```

## 九、监控与维护

### 1. 数据库监控指标
```bash
# 监控连接数
SHOW STATUS LIKE 'Threads_connected';

# 监控查询缓存命中率
SHOW STATUS LIKE 'Qcache%';

# InnoDB缓冲池命中率
SHOW STATUS LIKE 'Innodb_buffer_pool_read%';
```

### 2. 定期维护任务
```sql
-- 每周优化表
OPTIMIZE TABLE orders, order_items;

-- 每月重建统计信息
ANALYZE TABLE goods, users;
```

## 十、ThinkPHP 特定优化

### 1. 模型优化
```php
// 模型关联预加载
$orders = Order::with(['items', 'user', 'address'])
    ->where('status', 1)
    ->select();

// 替代N+1查询
// 错误方式：
$orders = Order::all();
foreach ($orders as $order) {
    $items = $order->items; // 每次循环都查询数据库
}

// 正确方式：
$orders = Order::with('items')->select();
```

### 2. 查询构造器优化
```php
// 使用field限制查询字段
Db::name('goods')
    ->field('id,goods_name,shop_price,main_image')
    ->where('is_on_sale', 1)
    ->select();

// 使用cache方法
Db::name('goods')
    ->where('category_id', $categoryId)
    ->cache(true, 3600, 'goods_category_' . $categoryId)
    ->select();
```

### 3. 批量操作优化
```php
// 批量插入替代循环插入
$data = [];
for ($i = 0; $i < 100; $i++) {
    $data[] = [
        'name' => 'goods_' . $i,
        'price' => rand(100, 1000)
    ];
}
Db::name('goods')->insertAll($data);

// 批量更新
Db::name('users')
    ->where('score', '<', 100)
    ->update(['level' => 1]);
```

## 总结

本方案针对电商系统特点，提供了完整的 ThinkPHP + MySQL 调优方案：

1. **架构层面**：主从分离、分库分表
2. **索引优化**：合理设计复合索引
3. **SQL优化**：避免全表扫描、JOIN优化、分页优化
4. **配置调优**：MySQL参数与ThinkPHP配置
5. **事务控制**：减少锁竞争与死锁预防
6. **数据管理**：归档与分区策略
7. **缓存协同**：多级缓存与库存控制
8. **监控维护**：性能监控与定期维护

实施建议：
1. 先进行慢查询分析，优先优化最耗时的SQL
2. 索引优化效果立竿见影，应优先实施
3. 高并发场景重点优化库存相关操作
4. 定期进行数据库健康检查
5. 根据业务增长动态调整配置参数