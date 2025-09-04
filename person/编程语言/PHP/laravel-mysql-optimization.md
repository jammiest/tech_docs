# Laravel 电商系统 MySQL 调优方案

> 表结构在此：[电商系统核心表结构DDL](/person/数据库/MySQL/电商系统核心表结构DDL.md)

## 一、数据库架构优化

### 1. 读写分离配置

```php
// config/database.php
'mysql' => [
    'read' => [
        'host' => [
            env('DB_READ_HOST_1', '192.168.1.2'),
            env('DB_READ_HOST_2', '192.168.1.3'),
        ],
        'username' => env('DB_READ_USERNAME', 'read_user'),
        'password' => env('DB_READ_PASSWORD', 'password'),
    ],
    'write' => [
        'host' => env('DB_WRITE_HOST', '192.168.1.1'),
        'username' => env('DB_WRITE_USERNAME', 'write_user'),
        'password' => env('DB_WRITE_PASSWORD', 'password'),
    ],
    'sticky' => true, // 保证同请求内读写一致性
],
```

### 2. 分库分表策略

```php
// 订单表按用户ID哈希分表
class Order extends Model
{
    public function getTable()
    {
        $userId = $this->user_id ?? 0;
        $suffix = $userId % 10; // 分为10个表
        return 'orders_' . $suffix;
    }
}

// 商品表按分类ID分库
class Goods extends Model
{
    public function getConnectionName()
    {
        $categoryId = $this->category_id ?? 0;
        return 'goods_db_' . ($categoryId % 3);
    }
}
```

## 二、索引优化方案

### 1. 核心表索引设计

```php
// 商品表迁移文件
Schema::table('goods', function (Blueprint $table) {
    $table->index(['category_id', 'is_on_sale'], 'idx_category_status');
    $table->index(['brand_id', 'shop_price'], 'idx_brand_price');
    $table->index(['sales_count', 'comment_count'], 'idx_popularity');
});

// 订单表迁移文件
Schema::table('orders', function (Blueprint $table) {
    $table->index(['user_id', 'order_status'], 'idx_user_status');
    $table->index(['created_at', 'total_amount'], 'idx_time_amount');
    $table->index(['pay_status', 'shipping_status'], 'idx_process');
});

// 订单商品表迁移文件
Schema::table('order_items', function (Blueprint $table) {
    $table->index(['order_id', 'goods_id'], 'idx_order_goods');
    $table->index(['sku_id', 'created_at'], 'idx_sku_sales');
});
```

### 2. 复合索引设计原则

```php
// 商品搜索复合索引
Schema::table('goods', function (Blueprint $table) {
    $table->index([
        'category_id', 
        'is_on_sale', 
        'shop_price', 
        'sales_count'
    ], 'idx_search');
});

// 用户订单查询复合索引
Schema::table('orders', function (Blueprint $table) {
    $table->index([
        'user_id', 
        'order_status', 
        'created_at'
    ], 'idx_user_query');
});
```

## 三、查询优化实践

### 1. Eloquent 查询优化

```php
// 避免N+1查询
$orders = Order::with(['items.goods', 'user.address'])->limit(100)->get();

// 使用延迟关联优化分页
$orders = Order::select('orders.*')
    ->joinSub(
        Order::select('id')->orderBy('id', 'desc')->limit(100)->offset(200),
        'sub',
        'orders.id',
        '=',
        'sub.id'
    )->get();

// 只选择必要字段
$goods = Goods::select(['id', 'goods_name', 'shop_price', 'main_image'])
    ->where('is_on_sale', 1)
    ->get();
```

### 2. 复杂查询优化

```php
// 商品销售统计（使用查询构建器）
$salesStats = DB::table('order_items')
    ->select(
        'goods_id',
        DB::raw('SUM(goods_number) as sales_volume'),
        DB::raw('SUM(goods_number * goods_price) as sales_amount')
    )
    ->whereBetween('created_at', [$startDate, $endDate])
    ->groupBy('goods_id')
    ->orderBy('sales_volume', 'desc')
    ->limit(10)
    ->get();

// 使用原生SQL优化复杂统计
$monthlySales = DB::select("
    SELECT 
        DATE_FORMAT(created_at, '%Y-%m') AS month,
        COUNT(*) AS order_count,
        SUM(total_amount) AS sales_amount
    FROM orders
    WHERE created_at BETWEEN ? AND ?
    GROUP BY month
    ORDER BY month
", [$startDate, $endDate]);
```

## 四、数据库配置优化

### 1. my.cnf 关键配置

```ini
[mysqld]
# 内存配置
innodb_buffer_pool_size = 12G  # 总内存的50-70%
innodb_buffer_pool_instances = 8
innodb_log_file_size = 512M
innodb_log_buffer_size = 64M

# 连接配置
max_connections = 500
thread_cache_size = 50
table_open_cache = 4000

# InnoDB配置
innodb_file_per_table = ON
innodb_flush_method = O_DIRECT
innodb_read_io_threads = 8
innodb_write_io_threads = 4
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000

# 查询缓存（电商系统建议关闭）
query_cache_type = 0
query_cache_size = 0

# 事务隔离级别
transaction_isolation = READ-COMMITTED
```

### 2. Laravel 数据库配置

```php
// config/database.php
'connections' => [
    'mysql' => [
        'strict' => false, // 关闭严格模式
        'modes' => [
            'STRICT_TRANS_TABLES',
            'NO_ZERO_IN_DATE',
            'NO_ZERO_DATE',
            'ERROR_FOR_DIVISION_BY_ZERO',
            'NO_AUTO_CREATE_USER',
            'NO_ENGINE_SUBSTITUTION'
        ],
        'options' => extension_loaded('pdo_mysql') ? [
            PDO::ATTR_EMULATE_PREPARES => false, // 禁用预处理模拟
            PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => true,
        ] : [],
    ],
],
```

## 五、事务与锁优化

### 1. 事务优化实践

```php
// 减少事务范围
DB::transaction(function () use ($orderData, $items) {
    $order = Order::create($orderData);
    
    $order->items()->createMany($items);
    
    // 库存扣减单独处理（见库存优化部分）
    $this->inventoryService->decreaseStock($items);
    
    // 记录日志
    $order->logs()->create(['content' => '订单创建']);
}, 3); // 最多重试3次
```

### 2. 死锁预防策略

```php
// 统一获取锁的顺序
public function createOrder(array $items)
{
    // 按照商品ID排序获取锁
    $skuIds = collect($items)->pluck('sku_id')->sort()->values();
    
    foreach ($skuIds as $skuId) {
        Redis::lock("sku_lock:{$skuId}", 10)->block(5);
    }
    
    try {
        // 业务逻辑...
    } finally {
        foreach ($skuIds as $skuId) {
            Redis::lock("sku_lock:{$skuId}")->release();
        }
    }
}
```

## 六、数据归档与分区

### 1. 历史数据归档

```php
// 创建归档命令
Artisan::command('archive:orders', function () {
    $cutoffDate = now()->subYear();
    
    // 创建归档表
    Schema::create('orders_archive', function (Blueprint $table) {
        $table->bigIncrements('id');
        // 保持与原表相同结构
    });
    
    // 迁移数据
    DB::table('orders')
        ->where('created_at', '<', $cutoffDate)
        ->orderBy('id')
        ->chunkById(1000, function ($orders) {
            DB::table('orders_archive')->insert($orders->toArray());
        });
    
    // 删除已归档数据
    DB::table('orders')
        ->where('created_at', '<', $cutoffDate)
        ->delete();
});
```

### 2. 表分区实现

```php
// 订单表按时间范围分区
Schema::table('orders', function (Blueprint $table) {
    $table->partitionByRange(YEAR('created_at'))([
        ['name' => 'p2022', 'value' => 2023],
        ['name' => 'p2023', 'value' => 2024],
        ['name' => 'pmax', 'value' => MAXVALUE],
    ]);
});
```

## 七、缓存与数据库协同

### 1. 多级缓存策略

```php
// 商品详情获取逻辑
public function getGoodsDetail($id)
{
    return Cache::tags(['goods', "goods:{$id}"])
        ->remember("goods:detail:{$id}", 3600, function () use ($id) {
            return Goods::with(['skus', 'brand', 'category'])
                ->find($id)
                ->append(['spec_data_text']);
        });
}

// 商品修改时清除缓存
public function updateGoods($id, $data)
{
    $goods = Goods::find($id);
    $goods->update($data);
    
    Cache::tags(["goods:{$id}"])->flush();
    event(new GoodsUpdated($goods));
}
```

### 2. 库存缓存方案

```php
// 库存服务
class InventoryService
{
    public function getStock($skuId)
    {
        return Cache::lock("stock:{$skuId}:lock", 10)->block(5, function () use ($skuId) {
            return Cache::remember("stock:{$skuId}", 60, function () use ($skuId) {
                return GoodsSku::find($skuId)->stock;
            });
        });
    }
    
    public function decreaseStock($skuId, $quantity)
    {
        $lock = Cache::lock("stock:{$skuId}:lock", 10);
        
        try {
            $lock->block(5);
            
            DB::transaction(function () use ($skuId, $quantity) {
                $affected = DB::table('goods_skus')
                    ->where('id', $skuId)
                    ->where('stock', '>=', $quantity)
                    ->decrement('stock', $quantity);
                
                if ($affected === 0) {
                    throw new \Exception('库存不足');
                }
                
                Cache::forget("stock:{$skuId}");
            });
        } finally {
            $lock->release();
        }
    }
}
```

## 八、监控与维护

### 1. 慢查询监控

```php
// 安装调试工具
composer require barryvdh/laravel-debugbar --dev

// 自定义查询监听
DB::listen(function ($query) {
    if ($query->time > 1000) { // 超过1秒的查询
        Log::channel('slow_query')->warning('Slow query', [
            'sql' => $query->sql,
            'bindings' => $query->bindings,
            'time' => $query->time,
        ]);
    }
});
```

### 2. 定期维护任务

```php
// Kernel.php
protected function schedule(Schedule $schedule)
{
    $schedule->command('db:monitor')->hourly();
    $schedule->command('db:optimize')->weekly();
}

// 数据库监控命令
Artisan::command('db:monitor', function () {
    $status = DB::select('SHOW STATUS WHERE Variable_name LIKE "Threads_%"');
    $variables = DB::select('SHOW VARIABLES LIKE "%cache%"');
    
    $this->table(
        ['Variable', 'Value'],
        array_merge($status, $variables)
    );
});
```

## 九、Laravel 特有优化

### 1. 模型优化技巧

```php
// 使用模型缓存
class Goods extends Model
{
    use \Illuminate\Database\Eloquent\Cachable;
    
    protected static $cacheLifetime = 3600; // 1小时
    
    public static function findCached($id)
    {
        return static::cacheFor(static::$cacheLifetime)->find($id);
    }
}

// 使用访问器缓存计算字段
class Order extends Model
{
    protected $appends = ['status_text'];
    
    public function getStatusTextAttribute()
    {
        return Cache::remember("order:{$this->id}:status_text", 60, function () {
            return $this->calculateStatusText();
        });
    }
}
```

### 2. 查询构建器优化

```php
// 使用批量插入
DB::table('order_items')->insert($items);

// 使用批量更新
DB::table('users')
    ->where('last_login', '<', now()->subYear())
    ->update(['status' => 'inactive']);

// 使用原生索引提示
DB::table('orders')
    ->from(DB::raw('orders USE INDEX (idx_user_status)'))
    ->where('user_id', $userId)
    ->where('status', 'paid')
    ->get();
```

## 十、总结与实施建议

### 1. 实施步骤
1. **架构优化**：先配置读写分离和连接池
2. **索引优化**：分析慢查询添加合适索引
3. **查询优化**：重构复杂查询，避免N+1问题
4. **配置调优**：调整MySQL和Laravel配置
5. **缓存整合**：实现多级缓存策略
6. **监控上线**：部署监控系统观察效果

### 2. 关键指标监控
```php
// 数据库健康检查
$metrics = [
    '连接数利用率' => DB::select('SHOW STATUS WHERE Variable_name = "Threads_connected"')[0]->Value / 
                DB::select('SHOW VARIABLES LIKE "max_connections"')[0]->Value,
    '缓冲池命中率' => 1 - DB::select('SHOW STATUS WHERE Variable_name = "Innodb_buffer_pool_reads"')[0]->Value / 
                  DB::select('SHOW STATUS WHERE Variable_name = "Innodb_buffer_pool_read_requests"')[0]->Value,
    '查询缓存命中率' => DB::select('SHOW STATUS WHERE Variable_name = "Qcache_hits"')[0]->Value / 
                 (DB::select('SHOW STATUS WHERE Variable_name = "Qcache_hits"')[0]->Value + 
                  DB::select('SHOW STATUS WHERE Variable_name = "Qcache_inserts"')[0]->Value),
];
```

### 3. 常见问题解决方案
1. **连接数不足**：增加连接池大小，使用`mysqlnd`驱动
2. **慢查询问题**：使用`EXPLAIN`分析，添加适当索引
3. **死锁问题**：统一获取锁的顺序，减少事务范围
4. **缓存一致性问题**：使用标签缓存和事件驱动更新

本方案针对Laravel电商系统的特点，从架构设计到具体实现提供了全面的MySQL优化方案，可显著提升数据库性能和处理能力。