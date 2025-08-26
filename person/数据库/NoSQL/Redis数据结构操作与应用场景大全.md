# Redis 数据结构操作与应用场景大全

本文全面介绍 Redis 的各种数据结构及其操作命令，并结合实际应用场景，提供最佳实践和优化建议。

## 一、String（字符串）

### 基本操作
- `SET key value`：设置键值对
- `GET key`：获取键值
- `SETEX key seconds value`：设置带过期时间的键值
- `INCR key`：将键值递增1
- `DECR key`：将键值递减1
- `APPEND key value`：追加字符串
- `STRLEN key`：获取字符串长度

### 高级操作
- `MSET key1 value1 key2 value2`：批量设置键值
- `MGET key1 key2`：批量获取键值
- `SETNX key value`：键不存在时设置（分布式锁基础）
- `GETSET key new_value`：设置新值并返回旧值

### 应用场景
1. **缓存系统**：缓存HTML片段、API响应等
2. **计数器**：文章阅读量、用户点赞数等
3. **分布式锁**：`SET lock_key unique_value NX EX 10`
4. **Session存储**：存储用户会话信息

### 最佳实践
- 合理设置过期时间避免内存泄漏
- 大字符串考虑压缩后存储
- 计数器使用INCR/DECR保证原子性

## 二、Hash（哈希表）

### 基本操作
- `HSET key field value`：设置哈希字段值
- `HGET key field`：获取哈希字段值
- `HMSET key field1 value1 field2 value2`：批量设置字段
- `HMGET key field1 field2`：批量获取字段
- `HGETALL key`：获取所有字段和值
- `HDEL key field`：删除字段
- `HINCRBY key field increment`：字段值增加

### 应用场景
1. **用户信息存储**：存储用户对象属性
2. **商品属性**：存储商品的多维属性
3. **配置管理**：存储系统配置项
4. **购物车**：用户ID为key，商品ID为field，数量为value

### 最佳实践
- 字段数量不宜过多（建议不超过1000）
- 小哈希使用更高效（Redis优化了小型哈希存储）
- 考虑将大哈希拆分为多个小哈希

## 三、List（列表）

### 基本操作
- `LPUSH key value`：左侧插入元素
- `RPUSH key value`：右侧插入元素
- `LPOP key`：左侧弹出元素
- `RPOP key`：右侧弹出元素
- `LRANGE key start stop`：获取范围元素
- `LLEN key`：获取列表长度
- `LTRIM key start stop`：修剪列表

### 高级操作
- `BLPOP key timeout`：阻塞式左侧弹出
- `BRPOP key timeout`：阻塞式右侧弹出
- `RPOPLPUSH source destination`：原子性地移动元素

### 应用场景
1. **消息队列**：LPUSH+BRPOP实现简单队列
2. **最新消息**：存储最新的N条记录
3. **时间线**：用户动态时间线
4. **任务队列**：工作进程处理任务

### 最佳实践
- 控制列表长度避免内存问题
- 需要严格顺序时使用列表
- 考虑使用Stream替代复杂队列场景

## 四、Set（集合）

### 基本操作
- `SADD key member`：添加元素
- `SREM key member`：删除元素
- `SMEMBERS key`：获取所有元素
- `SISMEMBER key member`：判断元素是否存在
- `SCARD key`：获取集合大小
- `SPOP key`：随机弹出元素

### 集合运算
- `SINTER key1 key2`：交集
- `SUNION key1 key2`：并集
- `SDIFF key1 key2`：差集
- `SINTERSTORE destination key1 key2`：存储交集结果

### 应用场景
1. **标签系统**：文章标签、用户兴趣
2. **共同好友**：计算用户交集
3. **唯一IP记录**：统计独立访问IP
4. **抽奖系统**：随机获取用户

### 最佳实践
- 大数据集考虑使用SSCAN替代SMEMBERS
- 频繁集合运算考虑使用SINTERSTORE存储结果
- 纯数字集合更节省内存

## 五、Sorted Set（有序集合）

### 基本操作
- `ZADD key score member`：添加元素
- `ZRANGE key start stop`：按分数升序获取
- `ZREVRANGE key start stop`：按分数降序获取
- `ZSCORE key member`：获取元素分数
- `ZRANK key member`：获取元素排名（升序）
- `ZREVRANK key member`：获取元素排名（降序）
- `ZINCRBY key increment member`：增加元素分数

### 范围操作
- `ZRANGEBYSCORE key min max`：按分数范围获取
- `ZCOUNT key min max`：统计分数范围内元素数量
- `ZREMRANGEBYRANK key start stop`：按排名删除
- `ZREMRANGEBYSCORE key min max`：按分数范围删除

### 应用场景
1. **排行榜**：游戏积分榜、热搜榜
2. **延迟队列**：使用时间戳作为score
3. **范围查询**：价格区间商品
4. **优先级队列**：任务优先级处理

### 最佳实践
- 合理设置分数范围避免性能问题
- 大数据集考虑使用ZSCAN替代ZRANGE
- 频繁更新的排行榜考虑定期持久化

## 六、HyperLogLog

### 基本操作
- `PFADD key element`：添加元素
- `PFCOUNT key`：估算基数
- `PFMERGE destkey sourcekey`：合并多个HLL

### 应用场景
1. **UV统计**：独立访客数统计
2. **搜索词去重**：统计不同搜索词数量
3. **大规模数据集基数估算**

### 最佳实践
- 适用于允许少量误差的场景
- 内存消耗极低（每个HLL约12KB）
- 不能获取具体元素，只能估算数量

## 七、Bitmap（位图）

### 基本操作
- `SETBIT key offset value`：设置位
- `GETBIT key offset`：获取位
- `BITCOUNT key`：统计设置为1的位数
- `BITOP operation destkey key1 key2`：位运算

### 应用场景
1. **用户签到**：每日签到记录
2. **活跃用户统计**：按月统计活跃用户
3. **布隆过滤器**：实现存在性判断

### 最佳实践
- 适合二元状态记录
- 偏移量从0开始计算
- 大位图考虑分片存储

## 八、Geospatial（地理空间）

### 基本操作
- `GEOADD key longitude latitude member`：添加位置
- `GEOPOS key member`：获取位置坐标
- `GEODIST key member1 member2`：计算距离
- `GEORADIUS key longitude latitude radius unit`：查找范围内成员

### 应用场景
1. **附近的人**：查找附近用户
2. **位置服务**：商家位置查询
3. **配送范围**：计算配送范围内的商家

### 最佳实践
- 使用合适的地图坐标系（WGS84）
- 合理设置查询半径
- 大量位置数据考虑分片

## 九、Stream（流）

### 基本操作
- `XADD key * field value`：添加消息
- `XREAD COUNT n STREAMS key ID`：读取消息
- `XRANGE key start end`：范围查询
- `XGROUP CREATE key groupname ID`：创建消费者组

### 应用场景
1. **消息队列**：替代传统消息队列
2. **事件溯源**：记录系统状态变化
3. **日志收集**：收集和分发日志

### 最佳实践
- 为每个消费者组维护消费状态
- 合理设置消息保留策略
- 考虑使用ACK机制确保消息处理

## 十、事务与管道

### 事务操作
- `MULTI`：开始事务
- `EXEC`：执行事务
- `DISCARD`：取消事务
- `WATCH key`：监视键变化

### 管道操作
```python
pipe = redis.pipeline()
pipe.set('key1', 'value1')
pipe.get('key2')
pipe.execute()
```

### 应用场景
1. **批量操作**：减少网络往返时间
2. **原子性操作**：保证命令序列的原子性
3. **乐观锁**：WATCH实现CAS操作

### 最佳实践
- 事务中避免耗时操作
- 管道适合批量非依赖操作
- WATCH在高并发下可能引起重试

## 十一、Lua脚本

### 基本使用
```lua
-- 示例脚本
local key = KEYS[1]
local value = ARGV[1]
return redis.call('SET', key, value)
```

### 应用场景
1. **复杂原子操作**：Redis不直接支持的操作
2. **减少网络开销**：多个操作合并执行
3. **定制功能**：实现业务特定逻辑

### 最佳实践
- 脚本应保持简单和高效
- 避免长时间运行的脚本
- 使用SCRIPT LOAD和EVALSHA提高效率

## 十二、键管理

### 常用命令
- `KEYS pattern`：查找匹配的键（生产慎用）
- `SCAN cursor MATCH pattern`：安全地迭代键
- `DEL key`：删除键
- `EXPIRE key seconds`：设置过期时间
- `TTL key`：查看剩余时间
- `TYPE key`：查看键类型

### 最佳实践
- 生产环境避免使用KEYS命令
- 使用SCAN迭代大数据集
- 为缓存数据设置合理的过期时间
- 定期清理无用键

## 十三、性能优化建议

1. **数据结构选择**：根据场景选择最合适的数据结构
2. **内存优化**：
   - 使用适当的数据编码（小哈希、小列表等）
   - 考虑使用ziplist优化小数据集
3. **命令优化**：
   - 使用批量操作减少网络往返
   - 避免大键和大量小键
4. **持久化策略**：
   - RDB适合备份和快速恢复
   - AOF提供更好的持久性保证
5. **集群部署**：
   - 数据分片解决单机内存限制
   - 读写分离提高吞吐量

## 十四、监控与维护

1. **监控指标**：
   - 内存使用情况
   - 命令统计和延迟
   - 客户端连接数
2. **常用工具**：
   - redis-cli
   - redis-benchmark
   - RedisInsight
3. **维护建议**：
   - 定期备份数据
   - 监控慢查询
   - 合理设置最大内存和淘汰策略

## 总结

Redis提供了丰富的数据结构，可以满足各种场景下的需求：

1. **简单键值**：String
2. **对象存储**：Hash
3. **队列和栈**：List
4. **无序集合**：Set
5. **有序集合**：Sorted Set
6. **基数统计**：HyperLogLog
7. **位操作**：Bitmap
8. **地理位置**：Geospatial
9. **消息流**：Stream

根据具体业务场景选择合适的数据结构，结合事务、管道和Lua脚本等功能，可以构建出高性能的Redis应用。同时，注意监控和维护，确保Redis服务的稳定性和可靠性。