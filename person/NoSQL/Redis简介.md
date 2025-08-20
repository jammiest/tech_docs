# Redis

> 参考文档  
> 官方英文文档：<https://redis.io/documentation>  
> 官方中文文档：<http://www.redis.cn/documentation.html>  
> 中文文档[腾讯]：<https://cloud.tencent.com/developer/doc/1203>

# Redis 数据类型及操作详解

## 一、基础数据类型

### 1. String（字符串）
**特点**：二进制安全，最大512MB

**常用命令**：
```redis
SET key value [EX seconds|PX milliseconds] [NX|XX]
GET key
INCR key  # 原子递增
DECR key  # 原子递减
APPEND key value
STRLEN key
MSET key1 value1 key2 value2
MGET key1 key2
```

**应用场景**：
- 缓存数据
- 计数器
- 分布式锁

### 2. Hash（哈希）
**特点**：键值对集合，适合存储对象

**常用命令**：
```redis
HSET key field value
HGET key field
HGETALL key
HDEL key field
HEXISTS key field
HKEYS key
HVALS key
HINCRBY key field increment
```

**应用场景**：
- 存储用户信息
- 商品属性
- 配置信息

### 3. List（列表）
**特点**：双向链表，支持两端操作

**常用命令**：
```redis
LPUSH key value [value ...]
RPUSH key value [value ...]
LPOP key
RPOP key
LRANGE key start stop
LLEN key
LINDEX key index
LTRIM key start stop
```

**应用场景**：
- 消息队列
- 最新消息排行
- 记录日志

### 4. Set（集合）
**特点**：无序、唯一元素集合

**常用命令**：
```redis
SADD key member [member ...]
SREM key member [member ...]
SMEMBERS key
SCARD key
SISMEMBER key member
SINTER key [key ...]  # 交集
SUNION key [key ...]  # 并集
SDIFF key [key ...]   # 差集
```

**应用场景**：
- 标签系统
- 共同好友
- 唯一IP记录

### 5. Sorted Set（有序集合）
**特点**：元素关联score，按score排序

**常用命令**：
```redis
ZADD key score member [score member ...]
ZRANGE key start stop [WITHSCORES]
ZREVRANGE key start stop [WITHSCORES]
ZRANK key member
ZREM key member [member ...]
ZCARD key
ZCOUNT key min max
ZSCORE key member
```

**应用场景**：
- 排行榜
- 带权重的消息队列
- 范围查找

## 二、高级数据结构

### 1. Bitmaps
**特点**：位操作

**常用命令**：
```redis
SETBIT key offset value
GETBIT key offset
BITCOUNT key [start end]
BITOP operation destkey key [key ...]
```

**应用场景**：
- 用户签到
- 活跃用户统计
- 布隆过滤器

### 2. HyperLogLog
**特点**：基数统计，0.81%误差率

**常用命令**：
```redis
PFADD key element [element ...]
PFCOUNT key [key ...]
PFMERGE destkey sourcekey [sourcekey ...]
```

**应用场景**：
- UV统计
- 大规模去重计数

### 3. Geospatial
**特点**：地理位置信息存储

**常用命令**：
```redis
GEOADD key longitude latitude member [longitude latitude member ...]
GEODIST key member1 member2 [unit]
GEORADIUS key longitude latitude radius unit [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
```

**应用场景**：
- 附近的人
- 地理位置搜索
- 距离计算

## 三、数据操作最佳实践

### 1. 批量操作
```redis
# 管道(pipeline)示例
MULTI
SET key1 value1
SET key2 value2
INCR counter
EXEC
```

### 2. 过期控制
```redis
EXPIRE key seconds
TTL key
PEXPIRE key milliseconds
PERSIST key
```

### 3. 事务处理
```redis
WATCH key
MULTI
...命令队列...
EXEC  # 或DISCARD
```

### 4. Lua脚本
```redis
EVAL "return redis.call('GET', KEYS[1])" 1 mykey
```

## 四、数据类型选择指南

| 数据类型 | 适用场景 | 优势 | 限制 |
|---------|---------|------|------|
| String | 简单键值存储 | 操作简单 | 存储结构化数据效率低 |
| Hash | 对象存储 | 节省空间 | 无法单独设置字段过期 |
| List | 顺序数据 | 两端操作高效 | 随机访问性能差 |
| Set | 无序唯一集合 | 集合运算 | 无排序 |
| ZSet | 排序集合 | 范围查询 | 内存消耗较大 |
| Bitmap | 位操作 | 极省空间 | 只适用特定场景 |
| HyperLogLog | 基数统计 | 内存效率高 | 有误差 |
| Geo | 地理位置 | 专业功能 | 实现较复杂 |

## 五、性能优化建议

1. **合理选择数据结构**：根据业务需求选择最合适类型
2. **控制键值大小**：单个value不超过10KB
3. **批量操作**：使用MSET/MGET替代多次SET/GET
4. **避免大集合**：单个集合元素不超过5000个
5. **使用SCAN替代KEYS**：避免阻塞
6. **设置合理过期时间**：避免内存泄漏

Redis的高效使用关键在于根据具体业务场景选择最适合的数据类型和操作方式。


## 安装

环境：
操作系统：CentOS 7 minimal
工具：wget,gcc

1、下载解压编译

```shell
# 下载（需要已经安装wget）
wget http://download.redis.io/releases/redis-5.0.4.tar.gz

# sha256值校验
sha256sum redis-5.0.4.tar.gz

# 解压
tar zxf redis-5.0.4.tar.gz

# 移动并且进入目录
mv redis-5.0.4 /usr/local
cd /usr/local/redis-5.0.4

# 要清除所有生成的文件
make distclean

# 编译（需要已经安装gcc）
make
```

2、启动Redis服务器

```shell
/usr/local/redis-5.0.4/src/redis-server
```

3、内置的客户端连接服务端

```shell
src/redis-cli

-----------
redis> set foo bar
OK
redis> get foo
"bar"
------------
```