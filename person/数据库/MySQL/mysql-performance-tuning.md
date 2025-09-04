# MySQL调优

基于 users/orders/goods 表的数据库调优方案

## 现有表结构分析

### 当前表结构概览

users 表

```sql
CREATE TABLE `users` (
  `user_id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` varchar(255) NOT NULL,
  `phone` varchar(20) DEFAULT NULL,
  `status` tinyint NOT NULL DEFAULT '1',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `idx_username` (`username`),
  UNIQUE KEY `idx_email` (`email`),
  KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

goods 表

```sql
CREATE TABLE `goods` (
  `goods_id` int NOT NULL AUTO_INCREMENT,
  `goods_name` varchar(100) NOT NULL,
  `category_id` int NOT NULL,
  `price` decimal(10,2) NOT NULL,
  `stock` int NOT NULL DEFAULT '0',
  `description` text,
  `image_url` varchar(255) DEFAULT NULL,
  `is_on_sale` tinyint NOT NULL DEFAULT '1',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`goods_id`),
  KEY `idx_category` (`category_id`),
  KEY `idx_on_sale` (`is_on_sale`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

orders 表

```sql
CREATE TABLE `orders` (
  `order_id` int NOT NULL AUTO_INCREMENT,
  `order_no` varchar(32) NOT NULL,
  `user_id` int NOT NULL,
  `total_amount` decimal(10,2) NOT NULL,
  `payment_status` tinyint NOT NULL DEFAULT '0',
  `shipping_status` tinyint NOT NULL DEFAULT '0',
  `address` text NOT NULL,
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`order_id`),
  UNIQUE KEY `idx_order_no` (`order_no`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_user` FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

order_items 表

```sql
CREATE TABLE `order_items` (
  `item_id` int NOT NULL AUTO_INCREMENT,
  `order_id` int NOT NULL,
  `goods_id` int NOT NULL,
  `goods_name` varchar(100) NOT NULL,
  `price` decimal(10,2) NOT NULL,
  `quantity` int NOT NULL,
  `subtotal` decimal(10,2) NOT NULL,
  PRIMARY KEY (`item_id`),
  KEY `idx_order_id` (`order_id`),
  KEY `idx_goods_id` (`goods_id`),
  CONSTRAINT `fk_order` FOREIGN KEY (`order_id`) REFERENCES `orders` (`order_id`),
  CONSTRAINT `fk_goods` FOREIGN KEY (`goods_id`) REFERENCES `goods` (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## 性能问题诊断

### 通过慢查询日志发现的典型问题

```sql
# 常见慢查询示例
SELECT * FROM orders WHERE user_id = 100 AND payment_status = 1 ORDER BY created_at DESC;
SELECT * FROM goods WHERE is_on_sale = 1 AND category_id = 5 ORDER BY price DESC LIMIT 100,20;
SELECT COUNT(*) FROM orders WHERE user_id IN (SELECT user_id FROM users WHERE status = 1);
```

### EXPLAIN 分析发现的问题

1. 部分查询未使用索引（全表扫描）
2. 存在文件排序（Using filesort）
3. 临时表使用（Using temporary）
4. 不合理的子查询

## 综合调优方案

### 索引优化方案

新增复合索引

```sql
-- orders表增加复合索引
ALTER TABLE orders ADD INDEX idx_user_payment (user_id, payment_status);

-- goods表增加复合索引
ALTER TABLE goods ADD INDEX idx_category_sale_price (category_id, is_on_sale, price);

-- order_items表增加覆盖索引
ALTER TABLE order_items ADD INDEX idx_order_goods (order_id, goods_id, quantity);
```

优化现有索引

```sql
-- 调整orders表的created_at索引为复合索引
ALTER TABLE orders DROP INDEX idx_created_at;
ALTER TABLE orders ADD INDEX idx_user_created (user_id, created_at);
```

## SQL语句优化

优化分页查询

```sql
-- 原始慢查询
SELECT * FROM goods 
WHERE is_on_sale = 1 AND category_id = 5 
ORDER BY price DESC LIMIT 100,20;

-- 优化方案：使用延迟关联
SELECT g.* FROM goods g
INNER JOIN (
    SELECT goods_id FROM goods
    WHERE is_on_sale = 1 AND category_id = 5
    ORDER BY price DESC
    LIMIT 100,20
) AS tmp ON g.goods_id = tmp.goods_id
ORDER BY g.price DESC;
```

优化子查询

```sql
-- 原始慢查询
SELECT COUNT(*) FROM orders 
WHERE user_id IN (SELECT user_id FROM users WHERE status = 1);

-- 优化方案：改用JOIN
SELECT COUNT(*) FROM orders o
JOIN users u ON o.user_id = u.user_id
WHERE u.status = 1;
```

### 表结构优化

垂直拆分大字段

```sql
-- 将goods表的description拆分到新表
CREATE TABLE goods_details (
    goods_id INT PRIMARY KEY,
    description TEXT,
    FULLTEXT INDEX idx_desc (description),
    FOREIGN KEY (goods_id) REFERENCES goods(goods_id)
);

-- 迁移数据
INSERT INTO goods_details (goods_id, description)
SELECT goods_id, description FROM goods;

-- 原表删除大字段
ALTER TABLE goods DROP COLUMN description;
```

订单表归档策略

```sql
-- 创建历史订单表
CREATE TABLE orders_archive LIKE orders;
ALTER TABLE orders_archive ADD INDEX idx_archive_user (user_id, created_at);

-- 定期归档(每月执行)
INSERT INTO orders_archive 
SELECT * FROM orders 
WHERE created_at < DATE_SUB(CURRENT_DATE(), INTERVAL 6 MONTH);

DELETE FROM orders 
WHERE created_at < DATE_SUB(CURRENT_DATE(), INTERVAL 6 MONTH);
```

### 配置参数优化

my.cnf 关键参数调整

```ini
[mysqld]
# 缓冲池配置(根据服务器内存调整)
innodb_buffer_pool_size = 4G
innodb_buffer_pool_instances = 4

# 日志配置
innodb_log_file_size = 512M
innodb_log_buffer_size = 32M

# 连接配置
max_connections = 300
thread_cache_size = 50

# 查询缓存(通常建议关闭)
query_cache_type = 0
query_cache_size = 0

# 其他优化
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
```

### 读写分离架构

```ini
-- 主库配置
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog-format = ROW
binlog_row_image = FULL

-- 从库配置
[mysqld]
server-id = 2
read-only = 1
relay-log = mysql-relay-bin
log-slave-updates = 1
```

### 应用层改造

```java
// 写操作使用主库
@Master
public void createOrder(Order order) {
    orderMapper.insert(order);
}

// 读操作使用从库
@Slave
public List<Order> getUserOrders(Long userId) {
    return orderMapper.selectByUserId(userId);
}
```

### 监控与维护方案

定期维护任务

```sql
-- 每周执行
OPTIMIZE TABLE orders, order_items;

-- 每天执行
ANALYZE TABLE users, goods, orders, order_items;

-- 每小时检查
SHOW STATUS LIKE 'Threads_connected';
SHOW ENGINE INNODB STATUS;
```

关键监控指标

```sql
-- 监控查询
SELECT * FROM sys.schema_index_statistics 
WHERE table_schema = DATABASE();

SELECT * FROM sys.statements_with_full_table_scans;

SELECT * FROM sys.statements_with_temp_tables;
```

慢查询分析脚本

```bash
#!/bin/bash
# 慢查询分析脚本
MYSQL_USER="root"
MYSQL_PASS="password"
LOG_FILE="/var/log/mysql/mysql-slow.log"

# 分析慢查询
pt-query-digest --user=$MYSQL_USER --password=$MYSQL_PASS $LOG_FILE > slow_report.txt

# 发送报告
mail -s "MySQL Slow Query Report" dba@example.com < slow_report.txt
```

## 预期优化效果

|优化项目	|优化前性能	|优化后预期|	提升幅度|
| --	| --	| -- |	-- |
|用户订单查询	|1200ms|	50ms|	24倍|
|商品分页查询	|800ms|	30ms|	26倍|
|订单创建TPS	|200|	500	|2.5倍|
|数据库CPU使用率	|80%|	40%	|降低50%|
|磁盘IOPS	|3000	|1000	|减少67%|

## 实施建议

1. 分阶段实施：
   - 第一阶段：索引优化+SQL改写
   - 第二阶段：配置优化+表结构调整
   - 第三阶段：架构优化(读写分离)
2. 变更窗口：
   - 索引添加：业务低峰期执行
   - 表结构变更：使用pt-online-schema-change工具
3. 回滚方案：
   - 备份关键表数据
   - 记录所有DDL操作
   - 准备索引删除语句
4. 效果验证：
   - 优化前后执行计划对比
   - 业务高峰期的性能监控
   - 慢查询日志数量统计

通过以上综合优化方案，可以显著提升基于users/orders/goods表的数据库性能，满足高并发电商系统的业务需求。