# ThinkPHP 多级缓存设计方案

一个基于ThinkPHP的多级缓存方案，包含以下几个层级：

> 基于 users/orders/goods/order_items 表的数据库，更详细表信息参见[数据库表结构设计](/person/数据库/README.md)

## 1. 缓存层级设计

### 1.1 一级缓存：请求级缓存（Request Cache）
- 生命周期：当前请求周期内有效
- 实现方式：使用PHP数组或ThinkPHP的request对象存储

### 1.2 二级缓存：本地缓存（Local Cache）
- 生命周期：当前进程/脚本执行期间有效
- 实现方式：ThinkPHP的Cache门面，使用文件或内存驱动

### 1.3 三级缓存：分布式缓存（Distributed Cache）
- 生命周期：自定义过期时间
- 实现方式：Redis/Memcached等高速缓存

### 1.4 四级缓存：数据库缓存（DB Cache）
- 生命周期：数据变更时失效
- 实现方式：MySQL查询缓存或ThinkPHP模型缓存

## 2. 具体实现方案

### 2.1 商品数据缓存示例

```php
namespace app\service;

use think\facade\Cache;
use app\model\Goods;

class GoodsService
{
    // 缓存前缀
    const CACHE_PREFIX = 'goods_';
    
    // 获取商品信息（带多级缓存）
    public function getGoodsInfo($goodsId)
    {
        // 一级缓存检查
        static $cache = [];
        if (isset($cache[$goodsId])) {
            return $cache[$goodsId];
        }
        
        // 二级缓存检查（本地缓存）
        $localKey = self::CACHE_PREFIX . 'local_' . $goodsId;
        if (Cache::store('file')->has($localKey)) {
            $data = Cache::store('file')->get($localKey);
            $cache[$goodsId] = $data; // 填充一级缓存
            return $data;
        }
        
        // 三级缓存检查（Redis）
        $redisKey = self::CACHE_PREFIX . $goodsId;
        $data = Cache::store('redis')->get($redisKey);
        if ($data) {
            // 填充二级缓存（本地缓存）
            Cache::store('file')->set($localKey, $data, 60);
            $cache[$goodsId] = $data; // 填充一级缓存
            return $data;
        }
        
        // 四级缓存：数据库查询
        $goods = Goods::withCache(60)->find($goodsId);
        if ($goods) {
            $data = $goods->toArray();
            
            // 填充各级缓存
            Cache::store('redis')->set($redisKey, $data, 3600); // 1小时
            Cache::store('file')->set($localKey, $data, 60); // 1分钟
            $cache[$goodsId] = $data; // 填充一级缓存
            
            return $data;
        }
        
        return null;
    }
    
    // 更新商品信息（同时清除缓存）
    public function updateGoodsInfo($goodsId, $data)
    {
        $goods = Goods::find($goodsId);
        if ($goods->save($data)) {
            // 清除多级缓存
            $this->clearGoodsCache($goodsId);
            return true;
        }
        return false;
    }
    
    // 清除商品缓存
    public function clearGoodsCache($goodsId)
    {
        // 清除一级缓存（通过刷新变量）
        static $cache = [];
        unset($cache[$goodsId]);
        
        // 清除二级缓存
        $localKey = self::CACHE_PREFIX . 'local_' . $goodsId;
        Cache::store('file')->delete($localKey);
        
        // 清除三级缓存
        $redisKey = self::CACHE_PREFIX . $goodsId;
        Cache::store('redis')->delete($redisKey);
        
        // 清除模型缓存
        Goods::destroyCache($goodsId);
    }
}
```

### 2.2 订单数据缓存示例

```php
namespace app\service;

use think\facade\Cache;
use app\model\Orders;
use app\model\OrderItems;

class OrderService
{
    const ORDER_CACHE_PREFIX = 'order_';
    const ORDER_ITEMS_CACHE_PREFIX = 'order_items_';
    
    // 获取订单详情（带多级缓存）
    public function getOrderDetail($orderId)
    {
        // 一级缓存检查
        static $cache = [];
        if (isset($cache[$orderId])) {
            return $cache[$orderId];
        }
        
        // 二级缓存检查（本地缓存）
        $localKey = self::ORDER_CACHE_PREFIX . 'local_' . $orderId;
        if (Cache::store('file')->has($localKey)) {
            $data = Cache::store('file')->get($localKey);
            $cache[$orderId] = $data;
            return $data;
        }
        
        // 三级缓存检查（Redis）
        $redisKey = self::ORDER_CACHE_PREFIX . $orderId;
        $data = Cache::store('redis')->get($redisKey);
        if ($data) {
            Cache::store('file')->set($localKey, $data, 60);
            $cache[$orderId] = $data;
            return $data;
        }
        
        // 四级缓存：数据库查询
        $order = Orders::withCache(60)->find($orderId);
        if ($order) {
            $data = $order->toArray();
            $data['items'] = $this->getOrderItems($orderId);
            
            // 填充各级缓存
            Cache::store('redis')->set($redisKey, $data, 1800); // 30分钟
            Cache::store('file')->set($localKey, $data, 60); // 1分钟
            $cache[$orderId] = $data;
            
            return $data;
        }
        
        return null;
    }
    
    // 获取订单商品项（单独缓存）
    protected function getOrderItems($orderId)
    {
        $redisKey = self::ORDER_ITEMS_CACHE_PREFIX . $orderId;
        $items = Cache::store('redis')->get($redisKey);
        
        if (!$items) {
            $items = OrderItems::where('order_id', $orderId)
                ->withCache(60)
                ->select()
                ->toArray();
            
            Cache::store('redis')->set($redisKey, $items, 1800);
        }
        
        return $items;
    }
    
    // 更新订单状态（同时清除缓存）
    public function updateOrderStatus($orderId, $statusData)
    {
        $order = Orders::find($orderId);
        if ($order->save($statusData)) {
            $this->clearOrderCache($orderId);
            return true;
        }
        return false;
    }
    
    // 清除订单缓存
    public function clearOrderCache($orderId)
    {
        // 清除一级缓存
        static $cache = [];
        unset($cache[$orderId]);
        
        // 清除二级缓存
        $localKey = self::ORDER_CACHE_PREFIX . 'local_' . $orderId;
        Cache::store('file')->delete($localKey);
        
        // 清除三级缓存
        $redisKey = self::ORDER_CACHE_PREFIX . $orderId;
        Cache::store('redis')->delete($redisKey);
        
        // 清除订单商品项缓存
        $itemsKey = self::ORDER_ITEMS_CACHE_PREFIX . $orderId;
        Cache::store('redis')->delete($itemsKey);
        
        // 清除模型缓存
        Orders::destroyCache($orderId);
    }
}
```

## 3. 缓存策略优化

### 3.1 缓存时间设置
- 高频访问数据：Redis缓存30分钟-1小时，本地缓存1分钟
- 低频访问数据：Redis缓存5-10分钟，本地缓存30秒
- 极少变更数据：Redis缓存24小时，本地缓存5分钟

### 3.2 缓存键设计
- 使用业务前缀防止冲突
- 包含ID和版本号（如数据变更时更新版本）
- 示例：`goods_v1_123`（商品v1版本ID为123的缓存）

### 3.3 缓存雪崩预防
- 对Redis缓存设置随机过期时间（如基础时间±随机值）
- 使用互斥锁防止缓存击穿

### 3.4 缓存更新策略
- 数据变更时主动清除相关缓存
- 设置适当的缓存过期时间作为兜底
- 对于关联数据，建立缓存依赖关系

## 4. 高级缓存技巧

### 4.1 批量获取缓存
```php
// 批量获取商品信息
public function getMultiGoodsInfo($goodsIds)
{
    $result = [];
    $uncachedIds = [];
    
    // 先检查本地缓存
    foreach ($goodsIds as $id) {
        $localKey = self::CACHE_PREFIX . 'local_' . $id;
        if (Cache::store('file')->has($localKey)) {
            $result[$id] = Cache::store('file')->get($localKey);
        } else {
            $uncachedIds[] = $id;
        }
    }
    
    // 批量从Redis获取未命中的数据
    if (!empty($uncachedIds)) {
        $redisKeys = array_map(function($id) {
            return self::CACHE_PREFIX . $id;
        }, $uncachedIds);
        
        $redisData = Cache::store('redis')->getMultiple($redisKeys);
        foreach ($uncachedIds as $index => $id) {
            if (isset($redisData[$index]) && $redisData[$index]) {
                $result[$id] = $redisData[$index];
                // 填充本地缓存
                $localKey = self::CACHE_PREFIX . 'local_' . $id;
                Cache::store('file')->set($localKey, $redisData[$index], 60);
            } else {
                // 从数据库查询单个缺失项
                $goods = Goods::withCache(60)->find($id);
                if ($goods) {
                    $result[$id] = $goods->toArray();
                    // 填充Redis和本地缓存
                    $redisKey = self::CACHE_PREFIX . $id;
                    Cache::store('redis')->set($redisKey, $result[$id], 3600);
                    $localKey = self::CACHE_PREFIX . 'local_' . $id;
                    Cache::store('file')->set($localKey, $result[$id], 60);
                }
            }
        }
    }
    
    return $result;
}
```

### 4.2 缓存预热
```php
// 商品缓存预热（定时任务）
public function preheatGoodsCache($limit = 100)
{
    // 获取热销商品ID列表
    $hotGoodsIds = Goods::where('is_on_sale', 1)
        ->order('sales_volume', 'desc')
        ->limit($limit)
        ->column('goods_id');
    
    // 批量预热缓存
    $this->getMultiGoodsInfo($hotGoodsIds);
}
```

### 4.3 延迟双删策略
```php
// 更新商品信息（延迟双删）
public function updateGoodsWithDelayDelete($goodsId, $data)
{
    $goods = Goods::find($goodsId);
    
    // 第一次删除缓存
    $this->clearGoodsCache($goodsId);
    
    // 执行数据库更新
    $result = $goods->save($data);
    
    if ($result) {
        // 延迟1秒后再次删除缓存（防止其他请求在更新期间写入旧缓存）
        \think\facade\Timer::after(1000, function() use ($goodsId) {
            $this->clearGoodsCache($goodsId);
        });
    }
    
    return $result;
}
```

## 5. 缓存监控与维护

1. **缓存命中率监控**：记录各级缓存的命中率
2. **缓存大小控制**：定期清理过期缓存
3. **缓存异常处理**：缓存服务不可用时自动降级
4. **缓存统计分析**：分析热点数据，优化缓存策略

这个多级缓存设计方案可以有效提升系统性能，减少数据库压力，同时保证数据的一致性。根据实际业务场景，可以调整各级缓存的存储时间和策略。