# Laravel 多级缓存设计方案

> 表结构在此：[电商系统核心表结构DDL](/person/数据库/MySQL/电商系统核心表结构DDL.md)

## 一、缓存架构设计

### 1. 多级缓存层级
```
用户请求
│
├── 浏览器缓存 (HTTP Cache)
├── CDN 缓存
├── 应用层缓存 (Redis)
├── 数据库缓存 (Query Cache)
└── 持久化存储 (MySQL)
```

### 2. 缓存策略矩阵

| 数据类型 | 缓存层级 | 缓存时间 | 更新策略 |
|---------|----------|----------|----------|
| 商品基本信息 | CDN + Redis | 24小时 | 商品修改时清除 |
| 商品详情页 | CDN + Redis | 1小时 | 价格/库存变更时清除 |
| 商品列表页 | Redis | 30分钟 | 新增/下架商品时清除 |
| 用户购物车 | Redis | 永久 | 实时更新 |
| 用户订单 | Redis + HTTP | 15分钟 | 订单状态变更时清除 |
| 商品库存 | Redis + 内存 | 5分钟 | 下单时原子更新 |

## 二、Laravel 缓存配置

### 1. 配置 `.env` 文件
```ini
CACHE_DRIVER=redis
REDIS_CLIENT=predis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
REDIS_CACHE_DB=1
```

### 2. 配置 `config/cache.php`
```php
'stores' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'cache',
        'lock_connection' => 'default',
    ],
    
    'goods' => [
        'driver' => 'redis',
        'connection' => 'goods',
        'prefix' => 'goods_cache:',
        'ttl' => 3600, // 1小时
    ],
    
    'user_data' => [
        'driver' => 'redis',
        'connection' => 'user',
        'prefix' => 'user_data:',
        'ttl' => 1800, // 30分钟
    ],
],
```

## 三、缓存服务实现

### 1. 商品缓存服务 `App\Services\GoodsCacheService`

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Cache;
use App\Models\Goods;
use App\Models\GoodsSku;

class GoodsCacheService
{
    const CACHE_PREFIX = 'goods:';
    const TTL = 3600; // 1小时
    
    // 获取商品基本信息
    public function getGoodsInfo($goodsId)
    {
        $cacheKey = self::CACHE_PREFIX . 'info:' . $goodsId;
        
        return Cache::store('goods')->remember($cacheKey, self::TTL, function() use ($goodsId) {
            $goods = Goods::with(['brand', 'category'])->find($goodsId);
            return $goods ? $goods->toArray() : null;
        });
    }
    
    // 获取商品SKU列表
    public function getGoodsSkus($goodsId)
    {
        $cacheKey = self::CACHE_PREFIX . 'skus:' . $goodsId;
        
        return Cache::store('goods')->remember($cacheKey, self::TTL, function() use ($goodsId) {
            return GoodsSku::where('goods_id', $goodsId)
                ->get()
                ->map(function($sku) {
                    return [
                        'id' => $sku->id,
                        'price' => $sku->price,
                        'stock' => $sku->stock,
                        'spec_data' => $sku->spec_data,
                        'spec_text' => $this->parseSpecText($sku->spec_data)
                    ];
                })
                ->toArray();
        });
    }
    
    // 清除商品缓存
    public function clearGoodsCache($goodsId)
    {
        Cache::store('goods')->forget(self::CACHE_PREFIX . 'info:' . $goodsId);
        Cache::store('goods')->forget(self::CACHE_PREFIX . 'skus:' . $goodsId);
        Cache::store('goods')->forget(self::CACHE_PREFIX . 'detail:' . $goodsId);
    }
    
    // 解析规格文本
    private function parseSpecText($specData)
    {
        // 实现规格JSON到文本的转换逻辑
        // ...
    }
}
```

### 2. 用户数据缓存服务 `App\Services\UserCacheService`

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Cache;
use App\Models\User;

class UserCacheService
{
    const CACHE_PREFIX = 'user:';
    const TTL = 1800; // 30分钟
    
    // 获取用户基本信息
    public function getUserInfo($userId)
    {
        $cacheKey = self::CACHE_PREFIX . 'info:' . $userId;
        
        return Cache::store('user_data')->remember($cacheKey, self::TTL, function() use ($userId) {
            return User::find($userId)?->toArray();
        });
    }
    
    // 获取用户购物车
    public function getUserCart($userId)
    {
        $cacheKey = self::CACHE_PREFIX . 'cart:' . $userId;
        
        return Cache::store('user_data')->remember($cacheKey, 0, function() use ($userId) {
            // 购物车数据通常需要实时性，但使用缓存减轻数据库压力
            return Cart::where('user_id', $userId)
                ->with('goods')
                ->get()
                ->toArray();
        });
    }
    
    // 更新用户购物车缓存
    public function updateUserCartCache($userId)
    {
        $cacheKey = self::CACHE_PREFIX . 'cart:' . $userId;
        Cache::store('user_data')->forget($cacheKey);
        $this->getUserCart($userId); // 重新加载
    }
}
```

## 四、Eloquent 模型缓存

### 1. 缓存查询结果 Trait `App\Traits\Cacheable`

```php
<?php

namespace App\Traits;

use Illuminate\Support\Facades\Cache;

trait Cacheable
{
    protected static $cacheEnabled = true;
    protected static $cacheTtl = 3600;
    
    public static function bootCacheable()
    {
        static::updated(function($model) {
            $model->clearCache();
        });
        
        static::deleted(function($model) {
            $model->clearCache();
        });
    }
    
    public function getCacheKey($suffix = '')
    {
        return get_class($model) . ':' . $this->getKey() . ($suffix ? ':'.$suffix : '');
    }
    
    public function clearCache()
    {
        Cache::forget($this->getCacheKey());
    }
    
    public static function cachedFind($id)
    {
        if (!static::$cacheEnabled) {
            return static::find($id);
        }
        
        $model = new static;
        $cacheKey = $model->getCacheKey();
        
        return Cache::remember($cacheKey, static::$cacheTtl, function() use ($id) {
            return static::find($id);
        });
    }
}
```

### 2. 在模型中使用

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use App\Traits\Cacheable;

class Goods extends Model
{
    use Cacheable;
    
    protected static $cacheTtl = 7200; // 2小时
    
    // 关联关系...
}
```

## 五、HTTP 缓存中间件

### 1. 商品详情页缓存 `App\Http\Middleware\CacheGoodsResponse`

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Cache;

class CacheGoodsResponse
{
    public function handle($request, Closure $next)
    {
        if ($request->isMethod('GET') && $request->route()->getName() === 'goods.detail') {
            $goodsId = $request->route('id');
            $cacheKey = 'response:goods:' . $goodsId;
            
            // 检查ETag
            $etag = $request->header('If-None-Match');
            $currentEtag = Cache::get($cacheKey . ':etag');
            
            if ($etag && $etag === $currentEtag) {
                return response()->json([], 304);
            }
            
            $response = $next($request);
            
            // 缓存成功的响应
            if ($response->getStatusCode() === 200) {
                $etag = md5($response->getContent());
                Cache::put($cacheKey, $response->getContent(), 3600);
                Cache::put($cacheKey . ':etag', $etag, 3600);
                $response->header('ETag', $etag);
                $response->header('Cache-Control', 'public, max-age=3600');
            }
            
            return $response;
        }
        
        return $next($request);
    }
}
```

## 六、Redis 缓存结构设计

### 1. 商品数据缓存结构
```
goods:info:[goodsId] - JSON格式商品基本信息
goods:skus:[goodsId] - JSON格式SKU列表
goods:detail:[goodsId] - 完整商品详情页HTML
goods:category:[categoryId] - 分类商品列表
```

### 2. 用户数据缓存结构
```
user:info:[userId] - JSON格式用户信息
user:cart:[userId] - JSON格式购物车数据
user:orders:[userId] - JSON格式订单列表
user:addresses:[userId] - JSON格式地址列表
```

### 3. 库存缓存结构
```
inventory:sku:[skuId] - SKU当前库存 (使用Redis原子操作)
inventory:lock:[skuId] - SKU库存锁 (防止超卖)
```

## 七、缓存更新策略

### 1. 事件监听器 `App\Listeners\GoodsEventSubscriber`

```php
<?php

namespace App\Listeners;

use App\Events\GoodsCreated;
use App\Events\GoodsUpdated;
use App\Events\GoodsDeleted;
use App\Services\GoodsCacheService;

class GoodsEventSubscriber
{
    protected $goodsCache;
    
    public function __construct(GoodsCacheService $goodsCache)
    {
        $this->goodsCache = $goodsCache;
    }
    
    public function handleCreated(GoodsCreated $event)
    {
        $this->goodsCache->clearGoodsCache($event->goodsId);
        $this->clearCategoryCache($event->categoryId);
    }
    
    public function handleUpdated(GoodsUpdated $event)
    {
        $this->goodsCache->clearGoodsCache($event->goodsId);
        $this->clearCategoryCache($event->categoryId);
    }
    
    public function handleDeleted(GoodsDeleted $event)
    {
        $this->goodsCache->clearGoodsCache($event->goodsId);
        $this->clearCategoryCache($event->categoryId);
    }
    
    protected function clearCategoryCache($categoryId)
    {
        Cache::forget('goods:category:' . $categoryId);
        Cache::forget('goods:category:tree');
    }
    
    public function subscribe($events)
    {
        $events->listen(
            GoodsCreated::class,
            [GoodsEventSubscriber::class, 'handleCreated']
        );
        
        $events->listen(
            GoodsUpdated::class,
            [GoodsEventSubscriber::class, 'handleUpdated']
        );
        
        $events->listen(
            GoodsDeleted::class,
            [GoodsEventSubscriber::class, 'handleDeleted']
        );
    }
}
```

### 2. 在 `EventServiceProvider` 中注册

```php
protected $listen = [
    GoodsCreated::class => [
        GoodsEventSubscriber::class,
    ],
    GoodsUpdated::class => [
        GoodsEventSubscriber::class,
    ],
    GoodsDeleted::class => [
        GoodsEventSubscriber::class,
    ],
];
```

## 八、库存缓存实现

### 1. 库存服务 `App\Services\InventoryService`

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Redis;
use App\Models\GoodsSku;

class InventoryService
{
    const CACHE_PREFIX = 'inventory:sku:';
    const LOCK_PREFIX = 'inventory:lock:';
    const LOCK_TIMEOUT = 5; // 秒
    
    // 获取SKU库存
    public function getStock($skuId)
    {
        $cacheKey = self::CACHE_PREFIX . $skuId;
        
        $stock = Redis::get($cacheKey);
        if ($stock === null) {
            $stock = GoodsSku::where('id', $skuId)->value('stock');
            Redis::set($cacheKey, $stock);
            Redis::expire($cacheKey, 300); // 5分钟
        }
        
        return (int)$stock;
    }
    
    // 减少库存
    public function decreaseStock($skuId, $quantity)
    {
        $lockKey = self::LOCK_PREFIX . $skuId;
        $cacheKey = self::CACHE_PREFIX . $skuId;
        
        // 获取分布式锁
        $lock = Redis::set($lockKey, 1, 'NX', 'EX', self::LOCK_TIMEOUT);
        if (!$lock) {
            throw new \Exception('获取库存锁失败');
        }
        
        try {
            // 使用Lua脚本保证原子性
            $script = "
                local stock = tonumber(redis.call('GET', KEYS[1]))
                local quantity = tonumber(ARGV[1])
                
                if stock >= quantity then
                    redis.call('DECRBY', KEYS[1], quantity)
                    return 1
                else
                    return 0
                end
            ";
            
            $result = Redis::eval($script, 1, $cacheKey, $quantity);
            
            if ($result === 0) {
                throw new \Exception('库存不足');
            }
            
            // 异步更新数据库
            dispatch(function() use ($skuId, $quantity) {
                GoodsSku::where('id', $skuId)
                    ->where('stock', '>=', $quantity)
                    ->decrement('stock', $quantity);
            });
            
            return true;
        } finally {
            Redis::del($lockKey);
        }
    }
    
    // 清除库存缓存
    public function clearStockCache($skuId)
    {
        Redis::del(self::CACHE_PREFIX . $skuId);
    }
}
```

## 九、前端缓存控制

### 1. Blade 模板中的缓存指令

```php
@cache('goods.detail.'.$goods->id, now()->addHours(1))
    <!-- 商品详情内容 -->
@endcache
```

### 2. 响应头控制中间件

```php
public function handle($request, Closure $next)
{
    $response = $next($request);
    
    if ($request->isMethod('GET')) {
        $maxAge = 3600; // 1小时
        
        if ($request->route()->getName() === 'goods.detail') {
            $response->header('Cache-Control', 'public, max-age='.$maxAge);
            $response->header('ETag', md5($response->getContent()));
        }
        
        if ($request->route()->getName() === 'goods.list') {
            $response->header('Cache-Control', 'public, max-age=1800'); // 30分钟
        }
    }
    
    return $response;
}
```

## 十、缓存预热与维护

### 1. 缓存预热命令 `App\Console\Commands\WarmUpCache`

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\Goods;
use App\Services\GoodsCacheService;

class WarmUpCache extends Command
{
    protected $signature = 'cache:warmup {--category= : 预热指定分类}';
    protected $description = '预热商品缓存';
    
    public function handle(GoodsCacheService $goodsCache)
    {
        $query = Goods::query()->with(['skus', 'brand', 'category']);
        
        if ($categoryId = $this->option('category')) {
            $query->where('category_id', $categoryId);
        }
        
        $count = 0;
        $query->chunk(100, function($goodsList) use ($goodsCache, &$count) {
            foreach ($goodsList as $goods) {
                $goodsCache->getGoodsInfo($goods->id);
                $goodsCache->getGoodsSkus($goods->id);
                $count++;
            }
            
            $this->info("已预热 {$count} 个商品缓存");
        });
        
        $this->info("缓存预热完成，共预热 {$count} 个商品");
    }
}
```

### 2. 定时任务配置

```php
$schedule->command('cache:warmup')->dailyAt('3:00');
$schedule->command('cache:clear-stale')->hourly();
```

## 十一、性能监控与调试

### 1. 缓存命中率监控

```php
public function cacheHitRate()
{
    $redis = Redis::connection();
    $stats = $redis->info('stats');
    
    $keyspaceHits = $stats['keyspace_hits'];
    $keyspaceMisses = $stats['keyspace_misses'];
    $total = $keyspaceHits + $keyspaceMisses;
    
    return $total > 0 ? ($keyspaceHits / $total * 100) : 0;
}
```

### 2. 调试工具栏集成

```php
// 在 AppServiceProvider 中注册
if ($this->app->environment('local')) {
    $this->app->register(\Barryvdh\Debugbar\ServiceProvider::class);
    
    // 添加自定义收集器
    Debugbar::addCollector(new CacheCollector());
}
```

## 十二、完整缓存流程示例

### 1. 商品详情控制器

```php
public function detail($id, GoodsCacheService $cache)
{
    // 检查HTTP缓存
    $etag = $cache->getGoodsEtag($id);
    if (request()->header('If-None-Match') === $etag) {
        return response()->json([], 304);
    }
    
    // 获取商品数据
    $goods = $cache->getGoodsInfo($id);
    $skus = $cache->getGoodsSkus($id);
    
    if (!$goods) {
        abort(404);
    }
    
    // 构建响应
    $response = response()->json([
        'goods' => $goods,
        'skus' => $skus,
        'related' => $cache->getRelatedGoods($id)
    ]);
    
    // 设置缓存头
    $response->header('Cache-Control', 'public, max-age=3600');
    $response->header('ETag', $etag);
    
    return $response;
}
```

### 2. 商品更新控制器

```php
public function update($id, Request $request, GoodsCacheService $cache)
{
    $goods = Goods::findOrFail($id);
    
    // 更新商品
    $goods->update($request->all());
    
    // 清除缓存
    $cache->clearGoodsCache($id);
    event(new GoodsUpdated($id, $goods->category_id));
    
    return response()->json(['success' => true]);
}
```

## 总结

这个 Laravel 多级缓存实现方案提供了：

1. **分层缓存架构**：从浏览器到数据库的多级缓存
2. **智能缓存策略**：根据不同数据类型采用不同缓存策略
3. **自动缓存更新**：通过事件监听自动维护缓存一致性
4. **高性能库存管理**：Redis原子操作保证库存准确性
5. **完善的监控维护**：缓存预热、命中率监控等功能

实施建议：
1. 根据实际业务需求调整缓存时间
2. 在高并发场景下增加本地内存缓存层
3. 定期分析缓存命中率优化缓存策略
4. 重要业务数据考虑使用缓存降级策略