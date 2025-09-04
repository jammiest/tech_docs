# ThinkPHP 多级缓存设计方案

> 表结构在此：[电商系统核心表结构DDL](/person/数据库/MySQL/电商系统核心表结构DDL.md)

### 1. 多级缓存层级
```
用户请求
│
├── 浏览器缓存 (HTTP Cache)
├── CDN 缓存
├── 应用层缓存 (Redis)
├── 本地缓存 (文件/内存)
└── 数据库缓存 (MySQL Query Cache)
```

### 2. 缓存策略矩阵

| 数据类型 | 缓存层级 | 缓存时间 | 更新策略 |
|---------|----------|----------|----------|
| 商品基本信息 | Redis + 本地缓存 | 2小时 | 商品修改时清除 |
| 商品详情页 | CDN + Redis | 1小时 | 价格/库存变更时清除 |
| 商品列表页 | Redis | 30分钟 | 新增/下架商品时清除 |
| 用户购物车 | Redis | 永久 | 实时更新 |
| 用户订单 | Redis + HTTP | 15分钟 | 订单状态变更时清除 |
| 商品库存 | Redis + 内存 | 5分钟 | 下单时原子更新 |

## 二、ThinkPHP 缓存配置

### 1. 配置 `.env` 文件
```ini
# 缓存设置
CACHE_DRIVER=redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_SELECT=1
REDIS_TIMEOUT=0
REDIS_PERSISTENT=false

# 本地缓存设置
LOCAL_CACHE_DRIVER=file
LOCAL_CACHE_PATH=../runtime/cache/
```

### 2. 配置 `config/cache.php`
```php
return [
    // 默认缓存驱动
    'default' => env('cache.driver', 'redis'),
    
    // 缓存连接配置
    'stores' => [
        'file' => [
            'type' => 'file',
            'path' => env('local_cache.path', '../runtime/cache/'),
            'expire' => 3600,
        ],
        
        'redis' => [
            'type' => 'redis',
            'host' => env('redis.host', '127.0.0.1'),
            'port' => env('redis.port', 6379),
            'password' => env('redis.password', ''),
            'select' => env('redis.select', 1),
            'timeout' => env('redis.timeout', 0),
            'persistent' => env('redis.persistent', false),
            'prefix' => 'tp_cache:',
        ],
        
        // 商品专用缓存
        'goods' => [
            'type' => 'redis',
            'host' => env('redis.host', '127.0.0.1'),
            'port' => env('redis.port', 6379),
            'select' => 2, // 使用独立的DB
            'prefix' => 'goods_cache:',
        ],
    ],
    
    // 缓存前缀
    'prefix' => env('cache.prefix', 'tp_'),
];
```

## 三、缓存服务实现

### 1. 商品缓存服务 `app/service/GoodsCacheService.php`

```php
<?php
namespace app\service;

use think\facade\Cache;
use app\model\Goods;
use app\model\GoodsSku;

class GoodsCacheService
{
    const CACHE_PREFIX = 'goods:';
    
    /**
     * 获取商品基本信息
     */
    public function getGoodsInfo($goodsId)
    {
        $cacheKey = self::CACHE_PREFIX . 'info:' . $goodsId;
        
        // 先检查本地缓存
        $localCache = Cache::store('file')->get($cacheKey);
        if ($localCache) {
            return $localCache;
        }
        
        // 再检查Redis缓存
        $redisCache = Cache::store('goods')->get($cacheKey);
        if ($redisCache) {
            // 回填本地缓存
            Cache::store('file')->set($cacheKey, $redisCache, 300); // 5分钟
            return $redisCache;
        }
        
        // 查询数据库
        $goods = Goods::with(['brand', 'category'])
            ->find($goodsId)
            ->toArray();
        
        if ($goods) {
            // 写入Redis缓存
            Cache::store('goods')->set($cacheKey, $goods, 7200); // 2小时
            // 写入本地缓存
            Cache::store('file')->set($cacheKey, $goods, 300); // 5分钟
        }
        
        return $goods;
    }
    
    /**
     * 获取商品SKU列表
     */
    public function getGoodsSkus($goodsId)
    {
        $cacheKey = self::CACHE_PREFIX . 'skus:' . $goodsId;
        
        return Cache::store('goods')->remember($cacheKey, function() use ($goodsId) {
            return GoodsSku::where('goods_id', $goodsId)
                ->select()
                ->toArray();
        }, 7200); // 2小时
    }
    
    /**
     * 清除商品缓存
     */
    public function clearGoodsCache($goodsId)
    {
        $prefix = self::CACHE_PREFIX;
        $keys = [
            $prefix . 'info:' . $goodsId,
            $prefix . 'skus:' . $goodsId,
            $prefix . 'detail:' . $goodsId,
        ];
        
        // 清除Redis缓存
        Cache::store('goods')->delete($keys);
        
        // 清除本地缓存
        Cache::store('file')->delete($keys);
    }
}
```

### 2. 用户缓存服务 `app/service/UserCacheService.php`

```php
<?php
namespace app\service;

use think\facade\Cache;
use app\model\User;
use app\model\Cart;

class UserCacheService
{
    const CACHE_PREFIX = 'user:';
    
    /**
     * 获取用户信息
     */
    public function getUserInfo($userId)
    {
        $cacheKey = self::CACHE_PREFIX . 'info:' . $userId;
        
        return Cache::remember($cacheKey, function() use ($userId) {
            return User::find($userId)->toArray();
        }, 1800); // 30分钟
    }
    
    /**
     * 获取用户购物车
     */
    public function getUserCart($userId)
    {
        $cacheKey = self::CACHE_PREFIX . 'cart:' . $userId;
        
        return Cache::remember($cacheKey, function() use ($userId) {
            return Cart::with('goods')
                ->where('user_id', $userId)
                ->select()
                ->toArray();
        }, 0); // 永久缓存，通过事件更新
    }
    
    /**
     * 更新用户购物车缓存
     */
    public function updateUserCartCache($userId)
    {
        $cacheKey = self::CACHE_PREFIX . 'cart:' . $userId;
        Cache::delete($cacheKey);
        $this->getUserCart($userId); // 重新加载
    }
}
```

## 四、HTTP 缓存中间件

### 1. 商品详情缓存中间件 `app/middleware/GoodsCache.php`

```php
<?php
namespace app\middleware;

use think\facade\Cache;
use think\Response;

class GoodsCache
{
    public function handle($request, \Closure $next)
    {
        if ($request->isGet() && $request->route('goods_id')) {
            $goodsId = $request->route('goods_id');
            $cacheKey = 'http:goods:' . $goodsId;
            
            // 检查ETag
            $etag = $request->header('If-None-Match');
            $currentEtag = Cache::get($cacheKey . ':etag');
            
            if ($etag && $etag === $currentEtag) {
                return Response::create()->code(304);
            }
            
            $response = $next($request);
            
            if ($response->getCode() == 200) {
                $content = $response->getContent();
                $etag = md5($content);
                
                // 缓存响应
                Cache::set($cacheKey, $content, 3600);
                Cache::set($cacheKey . ':etag', $etag, 3600);
                
                $response->header([
                    'ETag' => $etag,
                    'Cache-Control' => 'public, max-age=3600',
                ]);
            }
            
            return $response;
        }
        
        return $next($request);
    }
}
```

### 2. 注册中间件 `app/middleware.php`

```php
return [
    // 全局中间件
    \app\middleware\GoodsCache::class,
];
```

## 五、缓存事件监听

### 1. 商品事件监听器 `app/listener/GoodsEventListener.php`

```php
<?php
namespace app\listener;

use app\service\GoodsCacheService;
use think\facade\Cache;

class GoodsEventListener
{
    protected $goodsCache;
    
    public function __construct(GoodsCacheService $goodsCache)
    {
        $this->goodsCache = $goodsCache;
    }
    
    public function onGoodsUpdate($goods)
    {
        $this->goodsCache->clearGoodsCache($goods->id);
        
        // 清除分类缓存
        Cache::delete('goods:category:' . $goods->category_id);
        Cache::delete('goods:category:tree');
    }
    
    public function subscribe($events)
    {
        $events->listen('GoodsUpdate', [$this, 'onGoodsUpdate']);
        $events->listen('GoodsDelete', [$this, 'onGoodsUpdate']);
    }
}
```

### 2. 注册事件监听 `app/event.php`

```php
return [
    'listen' => [
        'GoodsUpdate' => ['app\listener\GoodsEventListener'],
        'GoodsDelete' => ['app\listener\GoodsEventListener'],
    ],
];
```

## 六、库存缓存实现

### 1. 库存服务 `app/service/InventoryService.php`

```php
<?php
namespace app\service;

use think\facade\Cache;
use think\facade\Db;
use app\model\GoodsSku;

class InventoryService
{
    const CACHE_PREFIX = 'inventory:';
    const LOCK_PREFIX = 'lock:';
    const LOCK_TIMEOUT = 5; // 秒
    
    /**
     * 获取SKU库存
     */
    public function getStock($skuId)
    {
        $cacheKey = self::CACHE_PREFIX . 'sku:' . $skuId;
        
        // 先检查本地缓存
        $stock = Cache::store('file')->get($cacheKey);
        if ($stock !== null) {
            return (int)$stock;
        }
        
        // 检查Redis缓存
        $stock = Cache::get($cacheKey);
        if ($stock === null) {
            $stock = GoodsSku::where('id', $skuId)->value('stock');
            Cache::set($cacheKey, $stock, 300); // 5分钟
        }
        
        // 回填本地缓存
        Cache::store('file')->set($cacheKey, $stock, 60); // 1分钟
        
        return (int)$stock;
    }
    
    /**
     * 减少库存
     */
    public function decreaseStock($skuId, $quantity)
    {
        $lockKey = self::LOCK_PREFIX . 'sku:' . $skuId;
        $cacheKey = self::CACHE_PREFIX . 'sku:' . $skuId;
        
        // 获取分布式锁
        $lock = Cache::add($lockKey, 1, self::LOCK_TIMEOUT);
        if (!$lock) {
            throw new \Exception('系统繁忙，请重试');
        }
        
        try {
            // 使用Lua脚本保证原子性
            $script = <<<LUA
                local stock = tonumber(redis.call('GET', KEYS[1]))
                local quantity = tonumber(ARGV[1])
                
                if stock and stock >= quantity then
                    redis.call('DECRBY', KEYS[1], quantity)
                    return 1
                else
                    return 0
                end
LUA;
            
            $result = Cache::eval($script, [$cacheKey], [$quantity]);
            
            if ($result === 0) {
                throw new \Exception('库存不足');
            }
            
            // 异步更新数据库
            Db::name('goods_sku')
                ->where('id', $skuId)
                ->where('stock', '>=', $quantity)
                ->dec('stock', $quantity);
                
            // 更新本地缓存
            Cache::store('file')->set($cacheKey, $result, 60);
            
            return true;
        } finally {
            Cache::delete($lockKey);
        }
    }
}
```

## 七、缓存预热与维护

### 1. 缓存预热命令 `app/command/WarmUpCache.php`

```php
<?php
namespace app\command;

use think\console\Command;
use think\console\Input;
use think\console\Output;
use app\service\GoodsCacheService;

class WarmUpCache extends Command
{
    protected function configure()
    {
        $this->setName('cache:warmup')
            ->setDescription('预热商品缓存');
    }
    
    protected function execute(Input $input, Output $output)
    {
        $goodsCache = app(GoodsCacheService::class);
        $count = 0;
        
        Db::name('goods')
            ->where('is_on_sale', 1)
            ->chunk(100, function($goodsList) use ($goodsCache, &$count) {
                foreach ($goodsList as $goods) {
                    $goodsCache->getGoodsInfo($goods['id']);
                    $goodsCache->getGoodsSkus($goods['id']);
                    $count++;
                }
                
                $output->info("已预热 {$count} 个商品缓存");
            });
        
        $output->info("缓存预热完成，共预热 {$count} 个商品");
    }
}
```

### 2. 定时任务配置 `app/command.php`

```php
return [
    'cache:warmup' => 'app\command\WarmUpCache',
];
```

## 八、使用示例

### 1. 商品控制器 `app/controller/Goods.php`

```php
<?php
namespace app\controller;

use app\service\GoodsCacheService;
use think\facade\Cache;

class Goods
{
    public function detail($id, GoodsCacheService $goodsCache)
    {
        // 检查HTTP缓存
        $etag = Cache::get('goods:detail:etag:' . $id);
        if (request()->header('If-None-Match') === $etag) {
            return response('', 304);
        }
        
        // 获取商品数据
        $goods = $goodsCache->getGoodsInfo($id);
        $skus = $goodsCache->getGoodsSkus($id);
        
        if (!$goods) {
            return json(['error' => '商品不存在'], 404);
        }
        
        // 构建响应
        $response = json([
            'goods' => $goods,
            'skus' => $skus,
        ]);
        
        // 设置缓存头
        $etag = md5($response->getContent());
        $response->header([
            'ETag' => $etag,
            'Cache-Control' => 'public, max-age=3600',
        ]);
        
        return $response;
    }
    
    public function update($id)
    {
        $goods = Goods::find($id);
        $goods->save(input());
        
        // 触发缓存更新事件
        event('GoodsUpdate', $goods);
        
        return json(['success' => true]);
    }
}
```

### 2. 购物车控制器 `app/controller/Cart.php`

```php
<?php
namespace app\controller;

use app\service\UserCacheService;

class Cart
{
    public function index($userId, UserCacheService $userCache)
    {
        $cartItems = $userCache->getUserCart($userId);
        return json($cartItems);
    }
    
    public function add($userId, $skuId, $quantity)
    {
        // 添加购物车逻辑...
        
        // 更新缓存
        $userCache = app(UserCacheService::class);
        $userCache->updateUserCartCache($userId);
        
        return json(['success' => true]);
    }
}
```

## 九、总结

本方案为 ThinkPHP 电商系统提供了完整的多级缓存实现：

1. **缓存层级清晰**：从浏览器到数据库的多级缓存体系
2. **缓存策略灵活**：根据不同数据类型采用不同缓存策略
3. **自动缓存更新**：通过事件监听自动维护缓存一致性
4. **高性能库存管理**：Redis原子操作保证库存准确性
5. **完善的工具支持**：缓存预热、监控等功能

实施建议：
1. 根据实际业务需求调整缓存时间
2. 高并发场景可增加本地内存缓存层
3. 定期分析缓存命中率优化缓存策略
4. 重要业务数据考虑使用缓存降级策略
5. 监控Redis内存使用情况，防止内存溢出