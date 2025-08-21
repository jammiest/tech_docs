# MySQL一览

## 数据库表结构设计

### 用户表(users)

```sql
CREATE TABLE `users` (
  `user_id` int NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `email` varchar(100) NOT NULL COMMENT '邮箱',
  `password` varchar(255) NOT NULL COMMENT '密码(加密存储)',
  `phone` varchar(20) DEFAULT NULL COMMENT '手机号',
  `status` tinyint NOT NULL DEFAULT '1' COMMENT '状态(1-正常,0-禁用)',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `idx_username` (`username`),
  UNIQUE KEY `idx_email` (`email`),
  KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表';
```

### 商品表(goods)

```sql
CREATE TABLE `goods` (
  `goods_id` int NOT NULL AUTO_INCREMENT COMMENT '商品ID',
  `goods_name` varchar(100) NOT NULL COMMENT '商品名称',
  `category_id` int NOT NULL COMMENT '分类ID',
  `price` decimal(10,2) NOT NULL COMMENT '商品价格',
  `stock` int NOT NULL DEFAULT '0' COMMENT '库存数量',
  `description` text COMMENT '商品描述',
  `image_url` varchar(255) DEFAULT NULL COMMENT '商品图片',
  `is_on_sale` tinyint NOT NULL DEFAULT '1' COMMENT '是否上架(1-是,0-否)',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`goods_id`),
  KEY `idx_category` (`category_id`),
  KEY `idx_on_sale` (`is_on_sale`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品表';
```

### 订单表(orders)

```sql
CREATE TABLE `orders` (
  `order_id` int NOT NULL AUTO_INCREMENT COMMENT '订单ID',
  `order_no` varchar(32) NOT NULL COMMENT '订单编号',
  `user_id` int NOT NULL COMMENT '用户ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT '订单总金额',
  `payment_status` tinyint NOT NULL DEFAULT '0' COMMENT '支付状态(0-未支付,1-已支付,2-已退款)',
  `shipping_status` tinyint NOT NULL DEFAULT '0' COMMENT '物流状态(0-未发货,1-已发货,2-已收货)',
  `address` text NOT NULL COMMENT '收货地址',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`order_id`),
  UNIQUE KEY `idx_order_no` (`order_no`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_user` FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='订单表';
```

### 订单商品表(order_items)

```sql
CREATE TABLE `order_items` (
  `item_id` int NOT NULL AUTO_INCREMENT COMMENT '订单项ID',
  `order_id` int NOT NULL COMMENT '订单ID',
  `goods_id` int NOT NULL COMMENT '商品ID',
  `goods_name` varchar(100) NOT NULL COMMENT '商品名称(快照)',
  `price` decimal(10,2) NOT NULL COMMENT '购买价格',
  `quantity` int NOT NULL COMMENT '购买数量',
  `subtotal` decimal(10,2) NOT NULL COMMENT '小计金额',
  PRIMARY KEY (`item_id`),
  KEY `idx_order_id` (`order_id`),
  KEY `idx_goods_id` (`goods_id`),
  CONSTRAINT `fk_order` FOREIGN KEY (`order_id`) REFERENCES `orders` (`order_id`),
  CONSTRAINT `fk_goods` FOREIGN KEY (`goods_id`) REFERENCES `goods` (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='订单商品表';
```

## 示例数据

### 用户数据

```sql
INSERT INTO `users` (`username`, `email`, `password`, `phone`, `status`) VALUES
('john_doe', 'john@example.com', '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', '13800138001', 1),
('jane_smith', 'jane@example.com', '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', '13800138002', 1),
('bob_johnson', 'bob@example.com', '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', '13800138003', 0);
```

### 商品数据

```sql
INSERT INTO `goods` (`goods_name`, `category_id`, `price`, `stock`, `description`, `image_url`, `is_on_sale`) VALUES
('iPhone 13', 1, 5999.00, 100, 'Apple iPhone 13 128GB', 'https://example.com/iphone13.jpg', 1),
('MacBook Pro', 1, 12999.00, 50, 'Apple MacBook Pro 14-inch M1 Pro', 'https://example.com/mbp14.jpg', 1),
('小米电视', 2, 3999.00, 80, '小米电视4A 55英寸', 'https://example.com/mitv.jpg', 1),
('华为手表', 3, 1999.00, 120, '华为Watch GT 3', 'https://example.com/huawei-watch.jpg', 0);
```

### 订单数据

```sql
-- 订单1
INSERT INTO `orders` (`order_no`, `user_id`, `total_amount`, `payment_status`, `shipping_status`, `address`) VALUES
('ORD20230001', 1, 5999.00, 1, 1, '北京市海淀区中关村大街1号');

INSERT INTO `order_items` (`order_id`, `goods_id`, `goods_name`, `price`, `quantity`, `subtotal`) VALUES
(1, 1, 'iPhone 13', 5999.00, 1, 5999.00);

-- 订单2
INSERT INTO `orders` (`order_no`, `user_id`, `total_amount`, `payment_status`, `shipping_status`, `address`) VALUES
('ORD20230002', 2, 16998.00, 1, 0, '上海市浦东新区张江高科技园区');

INSERT INTO `order_items` (`order_id`, `goods_id`, `goods_name`, `price`, `quantity`, `subtotal`) VALUES
(2, 2, 'MacBook Pro', 12999.00, 1, 12999.00),
(2, 3, '小米电视', 3999.00, 1, 3999.00);
```

## CRUD 语句

### 创建(Create)

```sql
-- 创建新用户
INSERT INTO `users` (`username`, `email`, `password`, `phone`) 
VALUES ('alice_wang', 'alice@example.com', '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', '13800138004');

-- 添加新商品
INSERT INTO `goods` (`goods_name`, `category_id`, `price`, `stock`, `is_on_sale`) 
VALUES ('iPad Air', 1, 4799.00, 60, 1);

-- 创建新订单(使用事务)
START TRANSACTION;
INSERT INTO `orders` (`order_no`, `user_id`, `total_amount`, `address`) 
VALUES ('ORD20230003', 3, 1999.00, '广州市天河区体育西路');

SET @order_id = LAST_INSERT_ID();

INSERT INTO `order_items` (`order_id`, `goods_id`, `goods_name`, `price`, `quantity`, `subtotal`) 
VALUES (@order_id, 4, '华为手表', 1999.00, 1, 1999.00);

COMMIT;
```

### 读取(Read)

```sql
-- 查询单个用户信息
SELECT * FROM `users` WHERE `user_id` = 1;

-- 查询所有上架商品
SELECT `goods_id`, `goods_name`, `price` 
FROM `goods` 
WHERE `is_on_sale` = 1 
ORDER BY `price` DESC;

-- 查询用户订单及商品明细
SELECT o.`order_no`, o.`total_amount`, o.`created_at`,
       i.`goods_name`, i.`price`, i.`quantity`, i.`subtotal`
FROM `orders` o
JOIN `order_items` i ON o.`order_id` = i.`order_id`
WHERE o.`user_id` = 2;

-- 统计各类商品销量
SELECT g.`category_id`, SUM(i.`quantity`) AS total_sold
FROM `goods` g
JOIN `order_items` i ON g.`goods_id` = i.`goods_id`
GROUP BY g.`category_id`;

-- 分页查询用户列表
SELECT `user_id`, `username`, `email`, `created_at`
FROM `users`
WHERE `status` = 1
ORDER BY `created_at` DESC
LIMIT 10 OFFSET 0;
```

### 更新(Update)

```sql
-- 更新用户信息
UPDATE `users` 
SET `phone` = '13800138888', `updated_at` = NOW()
WHERE `user_id` = 1;

-- 调整商品价格
UPDATE `goods`
SET `price` = `price` * 0.9  -- 打9折
WHERE `category_id` = 1 AND `is_on_sale` = 1;

-- 更新订单状态(支付成功)
UPDATE `orders`
SET `payment_status` = 1,
    `updated_at` = NOW()
WHERE `order_no` = 'ORD20230002';

-- 库存扣减(使用事务)
START TRANSACTION;
UPDATE `goods`
SET `stock` = `stock` - 1
WHERE `goods_id` = 1 AND `stock` >= 1;

-- 检查影响行数
IF ROW_COUNT() = 0 THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = '库存不足';
END IF;
COMMIT;
```

### 删除(Delete)

```sql
-- 删除用户(标记禁用而非物理删除)
UPDATE `users`
SET `status` = 0, `updated_at` = NOW()
WHERE `user_id` = 3;

-- 物理删除测试订单(谨慎使用)
DELETE FROM `order_items`
WHERE `order_id` IN (
    SELECT `order_id` FROM `orders` 
    WHERE `user_id` = 3 AND `payment_status` = 0
);

DELETE FROM `orders`
WHERE `user_id` = 3 AND `payment_status` = 0;

-- 清理下架商品(使用事务)
START TRANSACTION;
-- 先备份
INSERT INTO `goods_archive`
SELECT * FROM `goods`
WHERE `is_on_sale` = 0 AND `stock` = 0;

-- 再删除
DELETE FROM `goods`
WHERE `is_on_sale` = 0 AND `stock` = 0;
COMMIT;
```

### 高级查询示例

```sql
-- 查询每个用户的订单数和总消费金额
SELECT u.`user_id`, u.`username`, 
       COUNT(o.`order_id`) AS order_count,
       SUM(o.`total_amount`) AS total_spent,
       MAX(o.`created_at`) AS latest_order
FROM `users` u
LEFT JOIN `orders` o ON u.`user_id` = o.`user_id`
GROUP BY u.`user_id`
HAVING order_count > 0
ORDER BY total_spent DESC;

-- 查询热销商品(销量前10)
SELECT g.`goods_id`, g.`goods_name`, 
       SUM(i.`quantity`) AS total_sold,
       SUM(i.`subtotal`) AS total_revenue
FROM `goods` g
JOIN `order_items` i ON g.`goods_id` = i.`goods_id`
GROUP BY g.`goods_id`
ORDER BY total_sold DESC
LIMIT 10;

-- 使用CASE WHEN的复杂查询
SELECT 
    u.`user_id`,
    u.`username`,
    COUNT(o.`order_id`) AS order_count,
    SUM(CASE WHEN o.`payment_status` = 1 THEN o.`total_amount` ELSE 0 END) AS paid_amount,
    SUM(CASE WHEN o.`payment_status` = 0 THEN o.`total_amount` ELSE 0 END) AS unpaid_amount
FROM `users` u
LEFT JOIN `orders` o ON u.`user_id` = o.`user_id`
GROUP BY u.`user_id`;
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

### 示例：订单支付处理

下面是一个完整的订单支付处理存储过程，包含了参数验证、事务控制、库存检查、支付处理等电商系统核心功能：

```sql
DELIMITER //

CREATE PROCEDURE `process_order_payment`(
    IN p_order_no VARCHAR(32),
    IN p_payment_amount DECIMAL(10,2),
    IN p_payment_method VARCHAR(20),
    OUT p_result_code INT,
    OUT p_result_message VARCHAR(255)
)
BEGIN
    /*
     * 订单支付处理存储过程
     * 功能：处理订单支付，包括金额验证、库存扣减、状态更新等
     * 参数：
     *   - p_order_no: 订单编号
     *   - p_payment_amount: 支付金额
     *   - p_payment_method: 支付方式(wechat/alipay/credit)
     * 输出：
     *   - p_result_code: 结果代码(0=成功, <0=失败)
     *   - p_result_message: 结果描述
     */
    
    DECLARE v_order_id INT;
    DECLARE v_user_id INT;
    DECLARE v_order_total DECIMAL(10,2);
    DECLARE v_order_status INT;
    DECLARE v_payment_id INT;
    DECLARE v_item_count INT DEFAULT 0;
    DECLARE v_done INT DEFAULT FALSE;
    DECLARE v_goods_id INT;
    DECLARE v_quantity INT;
    
    -- 声明游标用于遍历订单商品
    DECLARE item_cursor CURSOR FOR 
        SELECT goods_id, quantity FROM order_items WHERE order_id = v_order_id;
    
    -- 声明异常处理
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_done = TRUE;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION 
    BEGIN
        ROLLBACK;
        SET p_result_code = -100;
        SET p_result_message = CONCAT('系统错误: ', COALESCE(ERROR_MESSAGE(), '未知错误'));
    END;
    
    -- 1. 验证订单信息
    SELECT order_id, user_id, total_amount, payment_status 
    INTO v_order_id, v_user_id, v_order_total, v_order_status
    FROM orders 
    WHERE order_no = p_order_no
    FOR UPDATE;
    
    IF v_order_id IS NULL THEN
        SET p_result_code = -1;
        SET p_result_message = '订单不存在';
        LEAVE proc_exit;
    END IF;
    
    IF v_order_status != 0 THEN
        SET p_result_code = -2;
        SET p_result_message = CONCAT('订单状态异常: ', 
            CASE v_order_status 
                WHEN 1 THEN '已支付' 
                WHEN 2 THEN '已退款' 
                ELSE '未知状态' 
            END);
        LEAVE proc_exit;
    END IF;
    
    -- 2. 验证支付金额
    IF p_payment_amount < v_order_total THEN
        SET p_result_code = -3;
        SET p_result_message = CONCAT('支付金额不足，需支付: ', v_order_total);
        LEAVE proc_exit;
    END IF;
    
    -- 3. 开始事务
    START TRANSACTION;
    
    -- 4. 检查并扣减库存
    OPEN item_cursor;
    item_loop: LOOP
        FETCH item_cursor INTO v_goods_id, v_quantity;
        IF v_done THEN
            LEAVE item_loop;
        END IF;
        
        -- 检查库存是否充足
        SET @v_current_stock = 0;
        SELECT stock INTO @v_current_stock 
        FROM goods 
        WHERE goods_id = v_goods_id 
        FOR UPDATE;
        
        IF @v_current_stock < v_quantity THEN
            SET p_result_code = -4;
            SET p_result_message = CONCAT('商品ID ', v_goods_id, ' 库存不足');
            CLOSE item_cursor;
            ROLLBACK;
            LEAVE proc_exit;
        END IF;
        
        -- 扣减库存
        UPDATE goods 
        SET stock = stock - v_quantity 
        WHERE goods_id = v_goods_id;
        
        SET v_item_count = v_item_count + 1;
    END LOOP;
    CLOSE item_cursor;
    
    -- 5. 记录支付信息(假设有payment_records表)
    INSERT INTO payment_records (
        order_id, 
        payment_no, 
        amount, 
        payment_method, 
        status
    ) VALUES (
        v_order_id,
        CONCAT('PAY', DATE_FORMAT(NOW(), '%Y%m%d%H%i%s'), 
        p_payment_amount, 
        p_payment_method, 
        1
    );
    
    SET v_payment_id = LAST_INSERT_ID();
    
    -- 6. 更新订单状态
    UPDATE orders 
    SET 
        payment_status = 1,
        payment_time = NOW(),
        updated_at = NOW()
    WHERE order_id = v_order_id;
    
    -- 7. 记录操作日志
    INSERT INTO order_logs (
        order_id, 
        action, 
        description, 
        operator
    ) VALUES (
        v_order_id,
        'payment',
        CONCAT('订单支付成功，金额: ', p_payment_amount, '，方式: ', p_payment_method),
        'system'
    );
    
    -- 8. 处理支付成功后的逻辑(如发送通知等)
    -- 这里可以调用其他存储过程或添加业务逻辑
    
    -- 提交事务
    COMMIT;
    
    -- 返回成功结果
    SET p_result_code = 0;
    SET p_result_message = CONCAT('支付成功，订单号: ', p_order_no, '，支付ID: ', v_payment_id);
    
    proc_exit: BEGIN END;
END //

DELIMITER ;
```

### 关键特性说明

完整的事务控制：

- 使用 `START TRANSACTION` 和 `COMMIT/ROLLBACK`
- 确保订单处理过程的原子性

全面的错误处理：

- 使用 `DECLARE EXIT HANDLER` 捕获异常
- 自定义错误码和错误消息
- 订单状态验证、金额验证、库存验证

游标使用：

- 使用游标遍历订单商品项
- 逐项检查并扣减库存

行级锁定：

- `FOR UPDATE` 锁定订单和商品记录
- 防止并发操作导致的数据不一致

业务逻辑封装：

- 支付记录创建
- 订单状态更新
- 操作日志记录
- 可扩展的通知逻辑

参数验证：

- 订单存在性检查
- 订单状态检查
- 支付金额验证
- 库存充足性检查

### 存储过程调用示例

```sql
-- 成功调用
SET @result_code = 0;
SET @result_message = '';
CALL process_order_payment('ORD20230001', 5999.00, 'wechat', @result_code, @result_message);
SELECT @result_code, @result_message;

-- 失败调用(金额不足)
CALL process_order_payment('ORD20230001', 5000.00, 'alipay', @result_code, @result_message);
SELECT @result_code, @result_message;

-- 失败调用(订单不存在)
CALL process_order_payment('ORD99999999', 5999.00, 'credit', @result_code, @result_message);
SELECT @result_code, @result_message;
```

### 实际应用场景

电商系统支付流程：

- 处理用户支付请求
- 确保库存和金额的准确性
- 记录完整的支付流水

订单状态管理：

- 统一管理订单状态转换
- 避免分散的业务逻辑

库存管理：

- 实时扣减库存
- 防止超卖

支付对账：

- 完整的支付记录
- 便于后续对账处理

这个存储过程展示了MySQL存储过程在复杂业务逻辑处理中的强大能力，适合作为电商系统核心支付流程的基础实现。