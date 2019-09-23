# Redis

> 参考文档  
> 官方英文文档：<https://redis.io/documentation>  
> 官方中文文档：<http://www.redis.cn/documentation.html>  
> 中文文档[腾讯]：<https://cloud.tencent.com/developer/doc/1203>

- select dbname ：选择数据库
- keys * ：查出redis数据库中所有 数据类型的数据
- exsits key ：查询这个key是否存在
- rename key key1：将key修改为key1（修改key的名字）
- type key：返回这个key的数据类型
- expire key seconds： 设置这个key的生存时间
- ttl key ： 查看这个key的剩余生存时间
- persist key ： 清除这个key的生存时间（永久化一个key）
- del key ：删除这个key，可以是任何类型
- randomkey：随机返回数据库中一个key
- quit ：退出redis客户端
- ctrl+c：强制退出redis客户端

## String字符串类型

> 主要应用于一些简单的数据存储

- set key value：往数据库中插入一个字符串数据
- mset key value ...：多条插入
- get key ：获取当前key
- mget key ...：获取多个key
- incr key：自增1
- incrby key index：自增指定值
- decr key ： 自减1
- decrby key index ：自减指定值
- del key ...：删除一个或多个

## map类型

> 主要应用于存储一个 实体参数信息，类似于java中new出来的bean实体对象

- hset key field value：往这个key中插入一个字段
- hmset key field value ...：往这个key中插入多个字段
- hget key field ：获取这个key的一个字段
- hmget key field ... ：获取这个key的多个字段
- hgetall key：获取这个key的所有字段
- hdel key field ...删除这个key的一个字段或多个字段
- hincrby key field index：自增指定数值（只能是整数类型数据）
- hexists key field：查找这个key中这个字段是否存在 （不存在返回0，存在返回1）
- hkeys key：获取这个key所有字段名称
- hvals key：获取这个key所有字段的值

## list 列表类型

> 主要应用于数据归类

- lpush key value ...：从左往右开始存储数据（可以存储多个value）
- rpush key value ...：从右往左存储数据（可以存储多个value）
- lrange key begin end：从左往右查出数据
- lpop key：从左往右取数据（取出来之后这条数据就不存在了）
- rpop key：从右往左取数据
- llen key：返回这个key的长度
- lrem key count value ：删除这个key中的这value值（count=0：删除所有对应这个value的值，count<0：从右边开始删，count>0：从左边开始删）

## set 无序集合类型

> （主要应用于抽奖类型的情况，因为这个集合是无序的）

- sadd key member ...：添加set集合数据
- spop key：随机取出这个key中的一个成员
- srem key member ...：删除这个key中所对应的成员（可以多个删除）
- smembers key：获取这个key中所有的成员
- sismember key member ：判断这个key中是否存在这个成员
- sdiff key1 key2 ...： 获取key1中key2没有的成员（可以多个）
- sinter key1 key2 ...：获取key1和key2都有的成员
- sunion key1 key2 ...：查出这两个key所有的成员，并合并相同的成员
- scard key： 获取这个key中的成员个数

## zset 有序集合类型

> （主要应用于销售额排行等情况）

- zadd key score member score member ...：有序添加
- zscore key member ：查询这个key中对应成员的分数
- zrem key member ：删除这个key中所对应的成员
- zrange key start stop：获取start到stop之间的索引值的所有成员（按分数从小到大返回，最后带withscores就会带分数一起查出来）
- zrevrange key start stop ：获取start到stop之间索引值的所有成员（按分数从大到小返回，最后带withscores就会带分数一起查出来）
- zrank key member：获取排名索引值（从小到大）
- zrevrank key member：获取排名索引值（从大到小）
- zcard key：获取这个key的成员个数
