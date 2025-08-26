# MySQL 错误码与信息全面解析

MySQL 使用特定的错误代码来标识各种运行时问题，以下是完整的错误码分类解析：

## 一、客户端错误 (1xxx-1999)

### 1.1 连接相关错误
| 错误码 | 常量名 | 描述 | 解决方案 |
|--------|--------|------|----------|
| 1045 | ER_ACCESS_DENIED_ERROR | 访问被拒绝 | 检查用户名/密码，确保有足够权限 |
| 1049 | ER_BAD_DB_ERROR | 未知数据库 | 确认数据库是否存在 |
| 2002 | CR_CONN_HOST_ERROR | 无法连接到服务器 | 检查MySQL服务状态和网络连接 |

### 1.2 SQL语法错误
```sql
-- 示例错误SQL
SELECT * FORM users;  -- 错误：FORM拼写错误
```
| 错误码 | 常量名 | 描述 | 常见原因 |
|--------|--------|------|----------|
| 1064 | ER_PARSE_ERROR | SQL语法错误 | 关键字拼写错误、缺少引号等 |
| 1146 | ER_NO_SUCH_TABLE | 表不存在 | 表名拼写错误或未创建 |

## 二、服务器错误 (2xxx-2999)

### 2.1 存储引擎错误
| 错误码 | 常量名 | 描述 | 相关存储引擎 |
|--------|--------|------|--------------|
| 1030 | ER_GET_ERRNO | 存储引擎错误 | MyISAM/InnoDB |
| 1213 | ER_LOCK_DEADLOCK | 死锁检测 | InnoDB |
| 1205 | ER_LOCK_WAIT_TIMEOUT | 锁等待超时 | InnoDB |

### 2.2 事务错误
```sql
-- 事务死锁示例
-- 会话1:
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- 会话2:
START TRANSACTION;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
UPDATE accounts SET balance = balance - 50 WHERE id = 1; -- 等待会话1
-- 会话1:
UPDATE accounts SET balance = balance + 50 WHERE id = 2; -- 死锁发生
```
| 错误码 | 描述 | 解决方案 |
|--------|------|----------|
| 1205 | 锁等待超时 | 增加`innodb_lock_wait_timeout`或优化事务 |
| 1213 | 死锁 | 重试事务或调整事务顺序 |

## 三、复制相关错误 (3xxx)

### 3.1 主从复制错误
| 错误码 | 常量名 | 描述 | 解决方案 |
|--------|--------|------|----------|
| 1236 | ER_MASTER_FATAL_ERROR_READING_BINLOG | 二进制日志读取错误 | 检查主库binlog文件 |
| 1594 | ER_SLAVE_FATAL_ERROR | 从库严重错误 | 检查从库状态`SHOW SLAVE STATUS` |

## 四、InnoDB 专用错误 (4xxx)

### 4.1 空间与配置错误
| 错误码 | 描述 | 相关参数 |
|--------|------|----------|
| 1114 | 表已满 | `innodb_data_file_path` |
| 1364 | 字段没有默认值 | `sql_mode=STRICT_TRANS_TABLES` |

## 五、常见错误处理示例

### 5.1 连接问题排查
```bash
# 检查MySQL服务状态
systemctl status mysqld

# 检查端口监听
netstat -tulnp | grep mysql

# 连接测试
mysql -u root -p -h 127.0.0.1
```

### 5.2 死锁分析
```sql
-- 查看最近死锁日志
SHOW ENGINE INNODB STATUS;

-- 关键信息查看
SELECT * FROM information_schema.INNODB_TRX;
SELECT * FROM performance_schema.events_statements_history;
```

## 六、错误日志配置

### 6.1 日志参数配置
```ini
# my.cnf 配置示例
[mysqld]
log_error = /var/log/mysql/error.log
log_warnings = 2
log_error_verbosity = 3
```

### 6.2 日志级别
| 级别 | 描述 | 对应参数 |
|------|------|----------|
| 1 | 错误日志 | log_error |
| 2 | 错误和警告 | log_warnings |
| 3 | 错误、警告和通知 | log_error_verbosity |

## 七、错误预防策略

### 7.1 SQL编写规范
```sql
-- 使用预处理语句防止注入
PREPARE stmt FROM 'SELECT * FROM users WHERE id = ?';
SET @id = 1;
EXECUTE stmt USING @id;
```

### 7.2 事务最佳实践
```sql
-- 设置合适的事务隔离级别
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 避免长事务
START TRANSACTION;
-- 快速操作
COMMIT;
```

## 八、性能相关错误

### 8.1 资源限制错误
| 错误码 | 描述 | 相关参数 |
|--------|------|----------|
| 1040 | 连接数过多 | `max_connections` |
| 1153 | 数据包太大 | `max_allowed_packet` |

### 8.2 查询优化建议
```sql
-- 使用EXPLAIN分析慢查询
EXPLAIN SELECT * FROM large_table WHERE condition;

-- 常见优化方案
ALTER TABLE orders ADD INDEX (customer_id);
ANALYZE TABLE orders;
```

## 九、错误信息获取方法

### 9.1 命令行查看
```sql
-- 查看最后错误代码
SELECT @@error_code;

-- 查看详细错误
SHOW ERRORS;
SHOW WARNINGS;
```

### 9.2 编程接口示例
```python
# Python示例
try:
    cursor.execute("INSERT INTO users VALUES (NULL)")
except mysql.connector.Error as err:
    print(f"错误代码: {err.errno}")
    print(f"SQL状态: {err.sqlstate}")
    print(f"错误消息: {err.msg}")
```

## 十、错误代码速查表

### 10.1 高频错误代码
| 代码 | 类型 | 频率 | 紧急程度 |
|------|------|------|----------|
| 1062 | 键值重复 | 高 | 中 |
| 1451 | 外键约束 | 高 | 高 |
| 2003 | 连接拒绝 | 中 | 高 |

### 10.2 错误处理优先级矩阵
$$
\text{处理优先级} = \begin{cases} 
\text{紧急} & \text{影响生产环境} \\
\text{高} & \text{影响部分功能} \\
\text{中} & \text{可规避问题} \\
\text{低} & \text{警告类信息}
\end{cases}
$$

通过系统性地理解MySQL错误码，可以快速定位和解决数据库问题。建议结合监控系统对关键错误进行实时告警，并建立错误处理知识库积累解决方案。