# Laravel 多级缓存设计方案

针对您提供的数据库结构，我将设计一个经典的多级缓存方案，包含以下几个层级：

1. **浏览器缓存** (HTTP缓存)
2. **应用层缓存** (Redis/Memcached)
3. **数据库缓存** (查询缓存)

## 1. 浏览器缓存 (HTTP缓存)

适用于不经常变化的资源，如商品图片、静态页面等。

```php
// 商品图片响应
public function getGoodsImage($goodsId)
{
    $goods = Goods::findOrFail($goodsId);
    
    return response()
        ->file(storage_path('app/public/' . $goods->image_url))
        ->setCache([
            'public' => true,
            'max_age' => 86400, // 1天
            'last_modified' => $goods->updated_at,
        ]);
}
```

## 2. 应用层缓存 (Redis)

### 2.1 缓存策略配置

在 `config/cache.php` 中配置 Redis：

```php
'stores' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'prefix' => 'laravel_cache:',
    ],
],
```

### 2.2 缓存键设计

```php
// 用户缓存键
const USER_CACHE_KEY = 'user:{id}';
// 商品缓存键
const GOODS_CACHE_KEY = 'goods:{id}';
// 订单缓存键
const ORDER_CACHE_KEY = 'order:{id}';
// 用户订单列表缓存键
const USER_ORDERS_CACHE_KEY = 'user_orders:{user_id}:{page}';
// 商品分类缓存键
const CATEGORY_GOODS_CACHE_KEY = 'category_goods:{category_id}:{page}';
```

### 2.3 缓存服务类

```php
namespace App\Services;

use Illuminate\Support\Facades\Cache;
use App\Models\{User, Goods, Order};

class CacheService
{
    // 用户缓存时间 (30分钟)
    const USER_CACHE_TTL = 1800;
    // 商品缓存时间 (1小时)
    const GOODS_CACHE_TTL = 3600;
    // 订单缓存时间 (15分钟)
    const ORDER_CACHE_TTL = 900;
    // 列表缓存时间 (5分钟)
    const LIST_CACHE_TTL = 300;
    
    /**
     * 获取用户信息 (带缓存)
     */
    public function getUser($userId)
    {
        $key = str_replace('{id}', $userId, self::USER_CACHE_KEY);
        
        return Cache::remember($key, self::USER_CACHE_TTL, function() use ($userId) {
            return User::find($userId);
        });
    }
    
    /**
     * 获取商品信息 (带缓存)
     */
    public function getGoods($goodsId)
    {
        $key = str_replace('{id}', $goodsId, self::GOODS_CACHE_KEY);
        
        return Cache::remember($key, self::GOODS_CACHE_TTL, function() use ($goodsId) {
            return Goods::with('category')->find($goodsId);
        });
    }
    
    /**
     * 获取订单信息 (带缓存)
     */
    public function getOrder($orderId)
    {
        $key = str_replace('{id}', $orderId, self::ORDER_CACHE_KEY);
        
        return Cache::remember($key, self::ORDER_CACHE_TTL, function() use ($orderId) {
            return Order::with('items')->find($orderId);
        });
    }
    
    /**
     * 获取用户订单列表 (带缓存)
     */
    public function getUserOrders($userId, $page = 1, $perPage = 15)
    {
        $key = str_replace(
            ['{user_id}', '{page}'], 
            [$userId, $page], 
            self::USER_ORDERS_CACHE_KEY
        );
        
        return Cache::remember($key, self::LIST_CACHE_TTL, function() use ($userId, $page, $perPage) {
            return Order::where('user_id', $userId)
                ->orderBy('created_at', 'desc')
                ->paginate($perPage, ['*'], 'page', $page);
        });
    }
    
    /**
     * 获取分类商品列表 (带缓存)
     */
    public function getCategoryGoods($categoryId, $page = 1, $perPage = 15)
    {
        $key = str_replace(
            ['{category_id}', '{page}'], 
            [$categoryId, $page], 
            self::CATEGORY_GOODS_CACHE_KEY
        );
        
        return Cache::remember($key, self::LIST_CACHE_TTL, function() use ($categoryId, $page, $perPage) {
            return Goods::where('category_id', $categoryId)
                ->where('is_on_sale', 1)
                ->orderBy('created_at', 'desc')
                ->paginate($perPage, ['*'], 'page', $page);
        });
    }
    
    /**
     * 清除用户缓存
     */
    public function clearUserCache($userId)
    {
        $key = str_replace('{id}', $userId, self::USER_CACHE_KEY);
        Cache::forget($key);
    }
    
    /**
     * 清除商品缓存
     */
    public function clearGoodsCache($goodsId)
    {
        $key = str_replace('{id}', $goodsId, self::GOODS_CACHE_KEY);
        Cache::forget($key);
        
        // 同时清除可能包含该商品的分类缓存
        $goods = Goods::find($goodsId);
        if ($goods) {
            $this->clearCategoryCache($goods->category_id);
        }
    }
    
    /**
     * 清除订单缓存
     */
    public function clearOrderCache($orderId)
    {
        $key = str_replace('{id}', $orderId, self::ORDER_CACHE_KEY);
        Cache::forget($key);
        
        // 同时清除用户订单列表缓存
        $order = Order::find($orderId);
        if ($order) {
            $this->clearUserOrdersCache($order->user_id);
        }
    }
    
    /**
     * 清除用户订单列表缓存
     */
    public function clearUserOrdersCache($userId)
    {
        // 使用通配符清除该用户所有分页的订单缓存
        $pattern = str_replace('{user_id}', $userId, self::USER_ORDERS_CACHE_KEY);
        $pattern = str_replace('{page}', '*', $pattern);
        
        $this->clearCacheByPattern($pattern);
    }
    
    /**
     * 清除分类商品列表缓存
     */
    public function clearCategoryCache($categoryId)
    {
        // 使用通配符清除该分类所有分页的商品缓存
        $pattern = str_replace('{category_id}', $categoryId, self::CATEGORY_GOODS_CACHE_KEY);
        $pattern = str_replace('{page}', '*', $pattern);
        
        $this->clearCacheByPattern($pattern);
    }
    
    /**
     * 根据模式清除缓存
     */
    protected function clearCacheByPattern($pattern)
    {
        if (config('cache.default') === 'redis') {
            // Redis 使用 scan 命令删除匹配的键
            $redis = Cache::getRedis();
            $iterator = null;
            
            do {
                $keys = $redis->scan($iterator, $pattern, 1000);
                if (!empty($keys)) {
                    $redis->del($keys);
                }
            } while ($iterator > 0);
        } else {
            // 其他驱动可能需要遍历所有键
            Cache::flush();
        }
    }
}
```

### 2.4 模型事件处理

在模型事件中自动更新缓存：

```php
// 在 AppServiceProvider 的 boot 方法中
User::updated(function ($user) {
    app(CacheService::class)->clearUserCache($user->user_id);
});

Goods::updated(function ($goods) {
    app(CacheService::class)->clearGoodsCache($goods->goods_id);
});

Order::updated(function ($order) {
    app(CacheService::class)->clearOrderCache($order->order_id);
});
```

## 3. 数据库缓存 (查询缓存)

对于复杂的查询，可以使用 Laravel 的查询缓存：

```php
// 使用 remember 方法缓存查询结果
$topSellingGoods = DB::table('order_items')
    ->select('goods_id', DB::raw('SUM(quantity) as total_sold'))
    ->groupBy('goods_id')
    ->orderBy('total_sold', 'desc')
    ->take(10)
    ->remember(now()->addHours(1)) // 缓存1小时
    ->get();
```

## 4. 缓存预热策略

在低峰期预先加载热门数据：

```php
// 在控制台命令中
class WarmUpCache extends Command
{
    protected $signature = 'cache:warmup';
    
    public function handle()
    {
        // 预热热门商品
        $popularGoodsIds = [1, 2, 3, 4, 5]; // 从配置或数据库中获取
        foreach ($popularGoodsIds as $id) {
            app(CacheService::class)->getGoods($id);
        }
        
        // 预热活跃用户
        $activeUserIds = User::where('status', 1)
            ->orderBy('last_login_at', 'desc')
            ->limit(100)
            ->pluck('user_id');
            
        foreach ($activeUserIds as $id) {
            app(CacheService::class)->getUser($id);
        }
    }
}
```

## 5. 缓存降级策略

当 Redis 不可用时，自动降级到文件缓存或直接查询数据库：

```php
// 修改 CacheService 类
protected function remember($key, $ttl, $callback)
{
    try {
        return Cache::store('redis')->remember($key, $ttl, $callback);
    } catch (\Exception $e) {
        // Redis 不可用时降级到文件缓存
        Log::error('Redis connection failed, fallback to file cache', ['error' => $e->getMessage()]);
        
        try {
            return Cache::store('file')->remember($key, $ttl, $callback);
        } catch (\Exception $e) {
            // 文件缓存也不可用时直接查询数据库
            Log::error('File cache failed, fallback to database', ['error' => $e->getMessage()]);
            return $callback();
        }
    }
}
```

## 6. 缓存使用示例

在控制器中使用缓存服务：

```php
class GoodsController extends Controller
{
    protected $cacheService;
    
    public function __construct(CacheService $cacheService)
    {
        $this->cacheService = $cacheService;
    }
    
    public function show($id)
    {
        $goods = $this->cacheService->getGoods($id);
        
        if (!$goods) {
            abort(404);
        }
        
        return view('goods.show', compact('goods'));
    }
}

class UserController extends Controller
{
    protected $cacheService;
    
    public function __construct(CacheService $cacheService)
    {
        $this->cacheService = $cacheService;
    }
    
    public function orders(Request $request)
    {
        $page = $request->get('page', 1);
        $orders = $this->cacheService->getUserOrders(auth()->id(), $page);
        
        return view('user.orders', compact('orders'));
    }
}
```

## 总结

这个多级缓存设计方案具有以下特点：

1. **分层缓存**：从浏览器到应用层再到数据库，逐层缓存
2. **智能失效**：通过模型事件自动更新相关缓存
3. **灵活配置**：可调整不同数据的缓存时间
4. **容错机制**：缓存降级策略保证系统可用性
5. **预热机制**：提前加载热门数据
6. **批量清除**：支持通配符清除相关缓存

通过这种设计，可以显著提高系统性能，减少数据库压力，同时保证数据的一致性。