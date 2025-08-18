# MySQL一览

## 表结构与示例数据

### 用户表(users)

```sql
CREATE TABLE `users` (
  `id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL,
  `email` varchar(100) NOT NULL,
  `password` varchar(255) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `status` enum('active','inactive','suspended') NOT NULL DEFAULT 'active',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_username` (`username`),
  UNIQUE KEY `idx_email` (`email`),
  KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- 示例数据
INSERT INTO `users` (`username`, `email`, `password`, `status`) VALUES
('john_doe', 'john@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'active'),
('jane_smith', 'jane@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'active'),
('bob_johnson', 'bob@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'inactive');
```

### 订单表(orders)

```sql
CREATE TABLE `orders` (
  `id` int NOT NULL AUTO_INCREMENT,
  `user_id` int NOT NULL,
  `order_number` varchar(20) NOT NULL,
  `amount` decimal(10,2) NOT NULL,
  `status` enum('pending','paid','shipped','delivered','cancelled') NOT NULL DEFAULT 'pending',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_order_number` (`order_number`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_status` (`status`),
  CONSTRAINT `fk_user` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- 示例数据
INSERT INTO `orders` (`user_id`, `order_number`, `amount`, `status`) VALUES
(1, 'ORD-1001', 99.99, 'paid'),
(1, 'ORD-1002', 149.99, 'shipped'),
(2, 'ORD-1003', 49.99, 'pending'),
(3, 'ORD-1004', 199.99, 'delivered');
```

## CRUD 语句

### 创建(Create)

```sql
-- 插入单条用户数据(忽略重复)
INSERT IGNORE INTO `users` (`username`, `email`, `password`) 
VALUES ('mike_wilson', 'mike@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi');

-- 批量插入订单数据(使用事务)
START TRANSACTION;
INSERT INTO `orders` (`user_id`, `order_number`, `amount`) VALUES
(1, 'ORD-1005', 79.99),
(2, 'ORD-1006', 129.99),
(2, 'ORD-1007', 59.99);
COMMIT;

-- 插入或更新(ON DUPLICATE KEY UPDATE)
INSERT INTO `users` (`username`, `email`, `password`) 
VALUES ('john_doe', 'john_new@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi')
ON DUPLICATE KEY UPDATE `email` = VALUES(`email`), `updated_at` = NOW();
```

### 读取(Read)

```sql
-- 基础查询(带条件、排序和分页)
SELECT `id`, `username`, `email`, `created_at` 
FROM `users` 
WHERE `status` = 'active' 
ORDER BY `created_at` DESC 
LIMIT 10 OFFSET 0;

-- 多表关联查询
SELECT u.`username`, o.`order_number`, o.`amount`, o.`status`
FROM `users` u
JOIN `orders` o ON u.`id` = o.`user_id`
WHERE u.`status` = 'active' AND o.`amount` > 50
ORDER BY o.`created_at` DESC;

-- 聚合查询
SELECT 
  u.`id`,
  u.`username`,
  COUNT(o.`id`) AS order_count,
  SUM(o.`amount`) AS total_amount
FROM `users` u
LEFT JOIN `orders` o ON u.`id` = o.`user_id`
GROUP BY u.`id`
HAVING order_count > 0
ORDER BY total_amount DESC;

-- 子查询
SELECT `username`, `email`
FROM `users`
WHERE `id` IN (
  SELECT DISTINCT `user_id` 
  FROM `orders` 
  WHERE `status` = 'paid' AND `amount` > 100
);
```

### 更新(Update)

```sql
-- 单表更新
UPDATE `users` 
SET `status` = 'inactive', `updated_at` = NOW()
WHERE `last_login` < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- 多表关联更新
UPDATE `orders` o
JOIN `users` u ON o.`user_id` = u.`id`
SET o.`status` = 'cancelled', o.`updated_at` = NOW()
WHERE u.`status` = 'suspended';

-- 条件更新(CASE WHEN)
UPDATE `products` 
SET `price` = CASE 
  WHEN `category` = 'premium' THEN `price` * 1.15
  WHEN `category` = 'standard' THEN `price` * 1.10
  ELSE `price` * 1.05
END,
`updated_at` = NOW();
```

### 删除(Delete)

```sql
-- 单表删除(带条件)
DELETE FROM `users` 
WHERE `status` = 'inactive' 
AND `created_at` < DATE_SUB(NOW(), INTERVAL 2 YEAR)
LIMIT 100;

-- 多表关联删除
DELETE o FROM `orders` o
JOIN `users` u ON o.`user_id` = u.`id`
WHERE u.`status` = 'suspended' AND o.`status` = 'pending';

-- 使用事务的安全删除
START TRANSACTION;
-- 先查询确认要删除的数据
SELECT * FROM `log_entries` WHERE `created_at` < DATE_SUB(NOW(), INTERVAL 1 YEAR);
-- 确认无误后执行删除
DELETE FROM `log_entries` WHERE `created_at` < DATE_SUB(NOW(), INTERVAL 1 YEAR);
COMMIT;
```

### 关键知识点总结

表设计：
- 主键、外键约束
- 适当的索引(唯一索引、普通索引)
- 字段类型选择(`ENUM`、`TIMESTAMP`等)
- 自动更新时间戳

CRUD操作：
- 事务处理(**ACID**特性)
- 批量操作提升性能
- `ON DUPLICATE KEY UPDATE` 实现upsert
- `JOIN`多表操作
- 子查询应用

性能优化：
- `LIMIT`分页控制
- `WHERE`条件使用索引字段
- 避免`SELECT *`
- 合理使用事务

安全考虑：
- 参数化查询防止SQL注入
- 操作前先查询确认
- 重要操作使用事务
- `DELETE`操作添加`LIMIT`

## 存储过程

```sql
DELIMITER //
-- 该存储过程包含了参数处理、事务控制、错误处理、条件逻辑等关键特性
CREATE PROCEDURE `process_order`(
    IN p_user_id INT,
    IN p_order_number VARCHAR(20),
    IN p_amount DECIMAL(10,2),
    OUT p_result_code INT,
    OUT p_result_message VARCHAR(100)
)
BEGIN
    /*
     * 订单处理存储过程
     * 功能：创建订单并处理支付，包含完整的业务逻辑验证
     * 参数：
     *   - p_user_id: 用户ID
     *   - p_order_number: 订单编号
     *   - p_amount: 订单金额
     * 输出：
     *   - p_result_code: 结果代码(0成功，非0失败)
     *   - p_result_message: 结果消息
     */
    
    DECLARE v_user_exists INT DEFAULT 0;
    DECLARE v_user_status VARCHAR(20);
    DECLARE v_order_id INT;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_code = -100;
        SET p_result_message = CONCAT('数据库错误: ', COALESCE(ERROR_MESSAGE(), '未知错误'));
    END;
    
    -- 验证用户是否存在且状态正常
    SELECT COUNT(*), `status` INTO v_user_exists, v_user_status
    FROM `users`
    WHERE `id` = p_user_id
    FOR UPDATE;
    
    IF v_user_exists = 0 THEN
        SET p_result_code = -1;
        SET p_result_message = '用户不存在';
        LEAVE proc_exit;
    END IF;
    
    IF v_user_status != 'active' THEN
        SET p_result_code = -2;
        SET p_result_message = CONCAT('用户状态异常: ', v_user_status);
        LEAVE proc_exit;
    END IF;
    
    -- 验证订单金额
    IF p_amount <= 0 THEN
        SET p_result_code = -3;
        SET p_result_message = '订单金额必须大于0';
        LEAVE proc_exit;
    END IF;
    
    -- 开始事务
    START TRANSACTION;
    
    -- 创建订单记录
    INSERT INTO `orders` (
        `user_id`,
        `order_number`,
        `amount`,
        `status`
    ) VALUES (
        p_user_id,
        p_order_number,
        p_amount,
        'pending'
    );
    
    SET v_order_id = LAST_INSERT_ID();
    
    -- 模拟支付处理
    IF p_amount > 1000 THEN
        -- 大额订单需要特殊处理
        UPDATE `orders` 
        SET `status` = 'pending_review'
        WHERE `id` = v_order_id;
        
        SET p_result_code = 1;
        SET p_result_message = '订单创建成功，等待人工审核';
    ELSE
        -- 正常订单直接标记为已支付
        UPDATE `orders` 
        SET `status` = 'paid'
        WHERE `id` = v_order_id;
        
        -- 更新用户最后下单时间(假设表中有此字段)
        UPDATE `users`
        SET `last_order_at` = NOW()
        WHERE `id` = p_user_id;
        
        SET p_result_code = 0;
        SET p_result_message = '订单创建并支付成功';
    END IF;
    
    -- 记录订单日志(假设有order_logs表)
    INSERT INTO `order_logs` (
        `order_id`,
        `action`,
        `description`
    ) VALUES (
        v_order_id,
        'create',
        CONCAT('订单创建，金额: ', p_amount)
    );
    
    -- 提交事务
    COMMIT;
    
    proc_exit: BEGIN END;
END //

DELIMITER ;
```

### 关键特性说明

参数处理：

- 输入参数(IN)：接收调用方传入的数据
- 输出参数(OUT)：返回处理结果
- 局部变量(DECLARE)：存储中间结果

事务控制：

- `START TRANSACTION` 开始事务
- `COMMIT` 提交事务
- `ROLLBACK` 回滚事务(在错误处理中)

错误处理：

- `DECLARE EXIT HANDLER FOR SQLEXCEPTION` 捕获SQL异常
- 自定义错误码和错误消息

业务逻辑验证：

- 用户存在性检查
- 用户状态检查
- 订单金额验证

条件处理：

- `IF-THEN-ELSE` 分支逻辑
- 大额订单特殊处理

数据一致性：

- `FOR UPDATE` 锁定用户记录
- 多表更新保证一致性

### 调用示例

```sql
-- 调用存储过程
CALL process_order(1, 'ORD-20230001', 99.99, @code, @message);

-- 查看结果
SELECT @code AS result_code, @message AS result_message;

-- 失败调用示例
CALL process_order(999, 'ORD-20230002', 0, @code, @message);
SELECT @code, @message;
```

### 实际应用场景

- 电商系统：订单创建、支付处理流程
- 银行系统：转账交易处理
- ERP系统：业务流程自动化
- 数据迁移：复杂数据转换逻辑封装

此存储过程展示了MySQL存储过程的核心功能，可以根据实际业务需求进行扩展和修改。