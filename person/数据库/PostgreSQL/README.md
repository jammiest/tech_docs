# PostgreSQL 数据库管理系统

PostgreSQL 是一个功能强大的开源关系型数据库管理系统(RDBMS)，以其可靠性、功能完整性和性能而著称。

## 核心特性

### 1. 数据类型支持
PostgreSQL 支持丰富的数据类型：

```sql
-- 基本类型
INTEGER, BIGINT, NUMERIC(10,2), VARCHAR(255), TEXT

-- 特殊类型
JSON, JSONB, XML, UUID, ARRAY, HSTORE

-- 几何类型
POINT, LINE, LBOX, PATH, POLYGON, CIRCLE

-- 网络地址类型
CIDR, INET, MACADDR
```

### 2. 高级功能

- **事务支持**：完全符合 ACID 特性
  - Atomicity（原子性）
  - Consistency（一致性）
  - Isolation（隔离性）
  - Durability（持久性）

- **MVCC（多版本并发控制）**：公式表示为：
  
  $$
  \text{Visibility} = \begin{cases}
  \text{true} & \text{if } \text{xmin} < \text{transaction_id} \leq \text{xmax} \\
  \text{false} & \text{otherwise}
  \end{cases}
  $$

- **窗口函数**：
  
  ```sql
  SELECT department, employee, salary,
         RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
  FROM employees;
  ```

### 3. 性能优化

| 优化技术 | 描述 | 适用场景 |
|---------|------|---------|
| 索引 | B-tree, Hash, GiST, SP-GiST, GIN, BRIN | 加速查询 |
| 分区 | 表分区 | 大表管理 |
| 并行查询 | 多核并行处理 | 复杂分析查询 |
| 物化视图 | 预计算视图 | 频繁访问的聚合数据 |

### 4. 扩展性

PostgreSQL 支持多种扩展：

```sql
-- 安装扩展示例
CREATE EXTENSION postgis;    -- 地理信息系统
CREATE EXTENSION hstore;     -- 键值存储
CREATE EXTENSION pg_trgm;    -- 文本相似度搜索
```

## 架构设计

PostgreSQL 采用多进程架构：

```
Client Application
    |
    v
Postmaster (主进程)
    |
    +-- Backend Process 1 (客户端连接1)
    +-- Backend Process 2 (客户端连接2)
    +-- Background Writer
    +-- WAL Writer
    +-- Autovacuum Launcher
    +-- ...其他辅助进程
```

## 配置优化

关键配置参数（postgresql.conf）：

```ini
# 内存相关
shared_buffers = 4GB                   # 25% of total RAM
work_mem = 16MB                        # 每个操作的内存
maintenance_work_mem = 512MB           # 维护操作内存

# 并行查询
max_worker_processes = 8
max_parallel_workers_per_gather = 4

# WAL配置
wal_level = replica
synchronous_commit = on
```

## 备份与恢复

1. **逻辑备份**：
   ```bash
   pg_dump -U username -d dbname -f backup.sql
   pg_restore -U username -d dbname backup.sql
   ```

2. **物理备份**：
   ```bash
   # 基础备份
   pg_basebackup -U username -D /backup/location -Ft -z -P
   
   # 时间点恢复(PITR)
   restore_command = 'cp /wal_archive/%f %p'
   recovery_target_time = '2023-01-01 12:00:00'
   ```

## 高可用方案

常见的高可用架构：

```
Primary PostgreSQL
    |
    v
Synchronous Standby (同步复制)
    |
    v
Asynchronous Standby (异步复制)
```

流复制配置：
```sql
-- 主库
ALTER SYSTEM SET wal_level = 'replica';
ALTER SYSTEM SET synchronous_commit = 'remote_apply';
ALTER SYSTEM SET synchronous_standby_names = 'standby1';

-- 备库
primary_conninfo = 'host=primary_host user=repl_user password=secret'
```

## 监控与维护

常用监控查询：

```sql
-- 连接数监控
SELECT datname, count(*) FROM pg_stat_activity GROUP BY datname;

-- 锁监控
SELECT blocked_locks.pid AS blocked_pid,
       blocking_locks.pid AS blocking_pid
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_locks blocking_locks
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid;
```

## 版本演进

| 版本 | 发布时间 | 重要特性 |
|------|----------|----------|
| 15 | 2022-10 | MERGE命令，逻辑复制改进 |
| 14 | 2021-09 | 并行计算增强，JSON便利性改进 |
| 13 | 2020-09 | 索引优化，并行清理 |
| 12 | 2019-10 | JIT编译优化，CTE改进 |

PostgreSQL 遵循每年发布一个大版本的节奏，每个版本都会带来显著的性能改进和新功能。