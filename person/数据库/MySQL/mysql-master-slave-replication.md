# MySQL 主从架构深度解析

MySQL主从架构是数据库系统中实现高可用性、负载均衡和数据备份的核心方案。以下是MySQL主从架构的全面分析：

## 1. 基础架构原理

### 1.1 架构组成
```plaintext
+----------------+       +----------------+
|   Master Node  | <---> |   Slave Node   |
| (Read/Write)   |       |  (Read Only)   |
+----------------+       +----------------+
```

### 1.2 数据同步流程
1. Master将变更写入二进制日志(binlog)
2. Slave的I/O线程从Master拉取binlog
3. Slave的SQL线程重放(apply)这些变更

### 1.3 关键公式
- 复制延迟计算：
  $$ \text{Replication Lag} = \text{Master当前时间} - \text{Slave最后应用的事务时间} $$

## 2. 主从复制模式

### 2.1 异步复制 (默认模式)
```sql
-- Master配置
[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_format = ROW
```

### 2.2 半同步复制
```sql
-- Master配置
[mysqld]
plugin-load = "rpl_semi_sync_master=semisync_master.so"
rpl_semi_sync_master_enabled = 1

-- Slave配置
[mysqld]
plugin-load = "rpl_semi_sync_slave=semisync_slave.so"
rpl_semi_sync_slave_enabled = 1
```

### 2.3 组复制 (Group Replication)
```sql
-- 组复制配置
[mysqld]
plugin-load = group_replication.so
group_replication_group_name = "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot = OFF
group_replication_local_address = "192.168.1.1:33061"
group_replication_group_seeds = "192.168.1.1:33061,192.168.1.2:33061"
```

## 3. 主从配置详解

### 3.1 Master节点配置
```sql
-- 创建复制用户
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

-- 查看Master状态
SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000003 | 107      |              |                  |
+------------------+----------+--------------+------------------+
```

### 3.2 Slave节点配置
```sql
-- 配置复制源
CHANGE MASTER TO
MASTER_HOST='master_host',
MASTER_USER='repl',
MASTER_PASSWORD='password',
MASTER_LOG_FILE='mysql-bin.000003',
MASTER_LOG_POS=107;

-- 启动复制
START SLAVE;

-- 查看复制状态
SHOW SLAVE STATUS\G
```

## 4. 主从架构拓扑

### 4.1 基础拓扑
```plaintext
Master -> Slave1
      -> Slave2
```

### 4.2 级联复制
```plaintext
Master -> Relay Slave -> Slave1
                    -> Slave2
```

### 4.3 环形复制
```plaintext
Node1 <-> Node2 <-> Node3
```

## 5. 性能优化策略

### 5.1 复制线程优化
```sql
-- 多线程复制配置
[mysqld]
slave_parallel_workers = 8
slave_parallel_type = LOGICAL_CLOCK
```

### 5.2 过滤复制
```sql
-- 只复制特定数据库
[mysqld]
replicate-do-db = db1
replicate-ignore-db = db2
```

### 5.3 延迟复制
```sql
-- 设置延迟时间(秒)
CHANGE MASTER TO MASTER_DELAY = 3600;
```

## 6. 高可用方案

### 6.1 故障转移流程
1. 检测Master故障
2. 提升Slave为New Master
3. 其他Slave重新指向New Master
4. 应用层切换连接

### 6.2 常见工具
- MHA (Master High Availability)
- Orchestrator
- MySQL Router

## 7. 监控指标

### 7.1 关键监控项
| 指标名称 | 计算公式 | 健康阈值 |
|---------|---------|---------|
| 复制延迟 | `Seconds_Behind_Master` | < 60s |
| IO线程状态 | `Slave_IO_Running` | Yes |
| SQL线程状态 | `Slave_SQL_Running` | Yes |
| 错误计数 | `Last_Errno` | 0 |

### 7.2 性能计算公式
- 复制吞吐量：
  $$ \text{Replication Throughput} = \frac{\text{Applied Transactions}}{\text{Time Interval}} $$

## 8. 架构设计建议

1. **网络拓扑**：
   - Master与Slave间网络延迟应 < 5ms
   - 建议带宽：$$ \text{带宽} \geq \text{峰值TPS} \times \text{平均事务大小} $$

2. **硬件配置**：
   - Slave硬件规格应不低于Master
   - 建议SSD存储，I/O性能公式：
     $$ \text{IOPS需求} = \text{峰值TPS} \times \text{平均每次事务IO操作数} $$

3. **容量规划**：
   - Slave存储空间公式：
     $$ \text{Slave存储} = \text{Master数据大小} \times (1 + \text{增长因子}) \times \text{保留天数} $$

MySQL主从架构是构建高可用数据库系统的基础，合理的设计和优化可以显著提升系统的可靠性和性能。