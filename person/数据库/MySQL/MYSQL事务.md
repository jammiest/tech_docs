# MYSQL事务篇

## 基本概念

ACID (`A`: atomicity,`C`: consistency,`I`: isolation,`D`: durability)

- `atomicity` **原子性** 一个事务的执行所有的操作，结果只有两种：要么全部执行、要么全部不执行。事务在执行的过程中，当在某一个节点执行发生错误的时候，事务会被执行rollback操作，将数据恢复到执行该事务之前的状态。A去银行转账，要么转账成功、要么转账失败。 
- `consistency` **一致性** 从事务开始执行到执行完成后，数据库的完整性约束完全没有收到破坏。A转账给B，不可能发生这种情况：A转账成功、B没有收到款。
- `isolation` **隔离性** 在同一时间点，数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交( *read uncommitted* )、读提交( *read committed* )、可重复读( *repeatable read*|默认方式 )和串行化( *serializable* )。
- `durability` **持久性** 事务成功执行后，事务的所有操作对数据库的更新是永久的，不能回滚。

## 隔离级别

- `READ UNCOMMITTED` **未提交读** 是指一个事务还没有提交时，它所做的变更能够被其他事务看到。
- `READ COMMITTED` **读已提交的** 是指一个事务只有在提交之后，它所做的变更才能被其他事务看到。
- `REPEATABLE READ` **可重复读** 是指一个事务中看到的数据始终是一致的。
- `SERIALIZABLE` **可串行化** 会为记录加锁，出现锁冲突时，后边的事务必须等待前边的事务执行完成才能继续。

### 隔离级别设置


| 事务隔离级别                    | 脏读 | 不可重复读 | 幻读 |
| ------------------------------- | ---- | ---------- | ---- |
| `READ UNCOMMITTED` **未提交读** | √   | √         | √   |
| `READ COMMITTED` **已提交读**   | ×   | √         | √   |
| `REPEATABLE READ` **可重复读**  | ×   | ×         | √ (标准) / ≈× (InnoDB)   |
| `SERIALIZABLE` **可串行化**     | ×   | ×         | ×   |

- **脏读**：`事务A`读取了`事务B`更新的数据，然后`事务B`回滚操作，那么`事务A`读取到的数据是脏数据
- **不可重复读**：`事务B`在`事务A`多次读取同一数据的过程中，对数据作了更新并提交，导致`事务A`读取同一数据结果不一致。
- **幻读**：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。

?> MySQL InnoDB 存储引擎的默认隔离级别是 REPEATABLE READ。可以通过以下命令查看：MySQL 8.0 之前：SELECT @@tx_isolation;MySQL 8.0 及之后：SELECT @@transaction_isolation;

```shell
mysql> SELECT @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
```

#### 系统配置隔离级别

```my.cnf
[mysqld]
transaction-isolation = REPEATABLE-READ
```


#### 通过SQL语句设置

> 语法

```sql
SET [GLOBAL | SESSION] TRANSACTION
    transaction_characteristic [, transaction_characteristic] ...

transaction_characteristic:
    ISOLATION LEVEL level
  | READ WRITE
  | READ ONLY

level:
     REPEATABLE READ
   | READ COMMITTED
   | READ UNCOMMITTED
   | SERIALIZABLE
```

> 语句

```sql
-- mysql 5.7
SELECT @@GLOBAL.transaction_isolation, @@transaction_isolation;
SET GLOBAL transaction_isolation='REPEATABLE-READ';
SET SESSION transaction_isolation='SERIALIZABLE';

-- mysql 5.6
SELECT @@GLOBAL.tx_isolation, @@tx_isolation;
SET GLOBAL tx_isolation='REPEATABLE-READ';
SET SESSION tx_isolation='SERIALIZABLE';
```

- 隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。
- 事务隔离级别为**读提交时**，写数据只会锁住相应的行
- 事务隔离级别为**串行化时**，读写数据都会锁住整张表


## 事务操作SQL
`START TRANSACTION |BEGIN`：显式地开启一个事务。

`COMMIT`：提交事务，使得对数据库做的所有修改成为永久性。

`ROLLBACK`：回滚会结束用户的事务，并撤销正在进行的所有未提交的修改。

## 解决幻读的方法
解决幻读的方式有很多，但是它们的核心思想就是一个事务在操作某张表数据的时候，另外一个事务不允许新增或者删除这张表中的数据了。解决幻读的方式主要有以下几种：将事务隔离级别调整为 SERIALIZABLE 。在可重复读的事务级别下，给事务操作的这张表添加表锁。在可重复读的事务级别下，给事务操作的这张表添加 Next-key Lock（Record Lock+Gap Lock）。