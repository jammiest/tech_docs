# Redis速查

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
SET key value [EX seconds|PX milliseconds] [NX|XX]  # 设置字符串键值，可设置过期时间和条件
GET key                                             # 获取指定键的字符串值
INCR key                                            # 将键的整数值原子性增加1
DECR key                                            # 将键的整数值原子性减少1
APPEND key value                                    # 在键的值后追加字符串
STRLEN key                                          # 获取键的字符串值的长度
MSET key1 value1 [key2 value2 ...]                  # 同时设置多个键值对
MGET key1 [key2 ...]                                # 同时获取多个键的值
```

**应用场景**：
- 缓存数据
- 计数器
- 分布式锁

### 2. Hash（哈希）
**特点**：键值对集合，适合存储对象

**常用命令**：
```redis
HSET key field value             # 设置哈希表中字段的值
HGET key field                   # 获取哈希表中字段的值
HGETALL key                      # 获取哈希表中所有字段和值
HDEL key field [field ...]       # 删除哈希表中的一个或多个字段
HEXISTS key field                # 判断哈希表中字段是否存在
HKEYS key                        # 获取哈希表中的所有字段名
HVALS key                        # 获取哈希表中的所有字段值
HINCRBY key field increment      # 为哈希表中的整数字段值增加增量
```

**应用场景**：
- 存储用户信息
- 商品属性
- 配置信息

### 3. List（列表）
**特点**：双向链表，支持两端操作

**常用命令**：
```redis
LPUSH key value [value ...]      # 将一个或多个值插入到列表头部（左边）
RPUSH key value [value ...]      # 将一个或多个值插入到列表尾部（右边）
LPOP key                         # 移除并返回列表的第一个元素（左边）
RPOP key                         # 移除并返回列表的最后一个元素（右边）
LRANGE key start stop            # 获取列表中指定范围内的元素
LLEN key                         # 获取列表的长度
LINDEX key index                 # 通过索引获取列表中的元素
LTRIM key start stop             # 修剪列表，只保留指定范围内的元素
```

**应用场景**：
- 消息队列
- 最新消息排行
- 记录日志

### 4. Set（集合）
**特点**：无序、唯一元素集合

**常用命令**：
```redis
SADD key member [member ...]      # 向集合中添加一个或多个成员
SREM key member [member ...]      # 从集合中移除一个或多个成员
SMEMBERS key                      # 获取集合中的所有成员
SCARD key                         # 获取集合的成员数量
SISMEMBER key member              # 判断成员是否在集合中
SINTER key [key ...]              # 返回多个集合的交集
SUNION key [key ...]              # 返回多个集合的并集
SDIFF key [key ...]               # 返回第一个集合与其他集合的差集
```

**应用场景**：
- 标签系统
- 共同好友
- 唯一IP记录

### 5. Sorted Set（有序集合）
**特点**：元素关联score，按score排序

**常用命令**：
```redis
ZADD key score member [score member ...]  # 向有序集合中添加成员及其分数
ZRANGE key start stop [WITHSCORES]        # 按分数升序返回索引范围内的成员
ZREVRANGE key start stop [WITHSCORES]     # 按分数降序返回索引范围内的成员
ZRANK key member                          # 返回成员在升序排列中的排名（从0开始）
ZREM key member [member ...]              # 从有序集合中移除一个或多个成员
ZCARD key                                 # 获取有序集合的成员数量
ZCOUNT key min max                        # 统计分数在[min,max]范围内的成员数量
ZSCORE key member                         # 获取指定成员的分数
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
SETBIT key offset value              # 设置或清除字符串值中指定偏移量上的位(bit)
GETBIT key offset                    # 获取字符串值中指定偏移量上的位(bit)
BITCOUNT key [start end]             # 统计字符串中被设置为1的比特位数量
BITOP operation destkey key [key ...] # 对多个键执行位运算并将结果存储到目标键
```

**应用场景**：
- 用户签到
- 活跃用户统计
- 布隆过滤器

### 2. HyperLogLog
**特点**：基数统计，0.81%误差率

**常用命令**：
```redis
PFADD key element [element ...]        # 向HyperLogLog添加元素（自动去重统计）
PFCOUNT key [key ...]                  # 返回HyperLogLog的基数估算值（去重后的元素数量）
PFMERGE destkey sourcekey [sourcekey ...] # 合并多个HyperLogLog到目标HyperLogLog
```

**应用场景**：
- UV统计
- 大规模去重计数

### 3. Geospatial
**特点**：地理位置信息存储

**常用命令**：
```redis
GEOADD key longitude latitude member [longitude latitude member ...]  # 添加地理空间位置（经度、纬度、名称）
GEODIST key member1 member2 [unit]                                    # 计算两个位置之间的距离
GEORADIUS key longitude latitude radius unit [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]  # 查找指定半径内的位置
```

**应用场景**：
- 附近的人
- 地理位置搜索
- 距离计算

## 三、数据操作最佳实践

### 1. 批量操作
```redis
# 管道(pipeline)示例
MULTI                      # 开始一个事务块（标记事务开始）
SET key1 value1           # 将命令放入队列（暂不执行）
SET key2 value2           # 将命令放入队列（暂不执行）
INCR counter              # 将命令放入队列（暂不执行）
EXEC                      # 执行所有事务块内的命令（原子性执行）
```

### 2. 过期控制
```redis
EXPIRE key seconds             # 为键设置过期时间（秒级精度）
TTL key                        # 查看键的剩余生存时间（秒）
PEXPIRE key milliseconds       # 为键设置过期时间（毫秒级精度）
PERSIST key                    # 移除键的过期时间，使其永久有效
```

### 3. 事务处理
```redis
WATCH key [key ...]           # 监视一个或多个键，如果事务执行前这些键被修改，则事务中断
MULTI                         # 开始事务块
...命令队列...                 # 将多个命令放入队列（如SET, GET, INCR等）
EXEC                          # 执行所有事务命令（如果WATCH的键未被修改）
DISCARD                       # 取消事务，清空命令队列，并解除WATCH
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