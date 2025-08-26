# 电商数据库百万级测试数据存储过程

## 一、批量用户数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_users(IN num_users INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE batch_size INT DEFAULT 1000;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    
    WHILE i <= num_users DO
        -- 批量插入用户数据
        INSERT INTO users (username, nickname, email, phone, password_hash, salt, gender, birthday, avatar, status, register_time, register_ip)
        SELECT 
            CONCAT('user', seq),
            CONCAT('用户', seq),
            CONCAT('user', seq, '@example.com'),
            CONCAT('138', LPAD(FLOOR(RAND() * 100000000), 8, '0')),
            '5f4dcc3b5aa765d61d8327deb882cf99',
            CONCAT('salt', FLOOR(RAND() * 1000)),
            FLOOR(RAND() * 3),
            DATE_ADD('1990-01-01', INTERVAL FLOOR(RAND() * 365*30) DAY),
            CONCAT('/avatars/user', seq, '.jpg'),
            1,
            DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 365*3) DAY),
            CONCAT('192.168.', FLOOR(RAND() * 255), '.', FLOOR(RAND() * 255))
        FROM (
            SELECT a.N + b.N * 10 + c.N * 100 + d.N * 1000 + i AS seq
            FROM (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) a
            CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) b
            CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) c
            CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) d
            LIMIT batch_size
        ) seq_table;
        
        SET i = i + batch_size;
        
        -- 每处理10000条输出一次进度
        IF i % 10000 = 1 THEN
            SELECT CONCAT('Generated ', i-1, ' users, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END WHILE;
    
    SELECT CONCAT('Total generated ', num_users, ' users, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 二、批量商品数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_goods(IN num_goods INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE batch_size INT DEFAULT 500;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    DECLARE category_count INT;
    DECLARE brand_count INT;
    
    SELECT COUNT(*) INTO category_count FROM goods_categories WHERE category_level = 2;
    SELECT COUNT(*) INTO brand_count FROM brands;
    
    WHILE i <= num_goods DO
        -- 批量插入商品数据
        INSERT INTO goods (category_id, goods_name, goods_sn, brand_id, market_price, shop_price, cost_price, stock, warn_stock, goods_weight, goods_brief, goods_desc, main_image, is_on_sale, is_recommend, is_new, is_hot, sales_count, comment_count, rating)
        SELECT 
            (SELECT id FROM goods_categories WHERE category_level = 2 ORDER BY RAND() LIMIT 1),
            CASE FLOOR(RAND() * 4)
                WHEN 0 THEN CONCAT('智能手机 ', seq, ' Pro')
                WHEN 1 THEN CONCAT('笔记本电脑 ', seq, ' Plus')
                WHEN 2 THEN CONCAT('智能家电 ', seq, ' Max')
                ELSE CONCAT('数码配件 ', seq)
            END,
            CONCAT('SN', DATE_FORMAT(NOW(), '%Y%m%d'), LPAD(seq, 6, '0')),
            (SELECT id FROM brands ORDER BY RAND() LIMIT 1),
            FLOOR(RAND() * 3000) + 1000,
            FLOOR(RAND() * 2000) + 800,
            FLOOR(RAND() * 1500) + 500,
            FLOOR(RAND() * 500) + 100,
            50,
            CASE FLOOR(RAND() * 4)
                WHEN 0 THEN 0.2
                WHEN 1 THEN 2.5
                WHEN 2 THEN 5.0
                ELSE 0.5
            END,
            CONCAT('商品', seq, '的简要描述'),
            CONCAT('<p>商品', seq, '的详细描述</p><p>功能特点：</p><ul><li>特点1</li><li>特点2</li></ul>'),
            CONCAT('/goods/main/', seq, '.jpg'),
            1,
            CASE WHEN RAND() > 0.7 THEN 1 ELSE 0 END,
            CASE WHEN RAND() > 0.7 THEN 1 ELSE 0 END,
            CASE WHEN RAND() > 0.7 THEN 1 ELSE 0 END,
            FLOOR(RAND() * 1000),
            FLOOR(RAND() * 500),
            ROUND(RAND() * 2 + 3, 1)
        FROM (
            SELECT a.N + b.N * 10 + c.N * 100 + d.N * 1000 + i AS seq
            FROM (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) a
            CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) b
            CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) c
            CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) d
            LIMIT batch_size
        ) seq_table;
        
        SET i = i + batch_size;
        
        -- 每处理5000条输出一次进度
        IF i % 5000 = 1 THEN
            SELECT CONCAT('Generated ', i-1, ' goods, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END WHILE;
    
    SELECT CONCAT('Total generated ', num_goods, ' goods, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 三、批量SKU数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_skus(IN skus_per_goods INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE goods_id_val BIGINT;
    DECLARE goods_count INT DEFAULT 0;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    
    -- 声明游标
    DECLARE goods_cursor CURSOR FOR SELECT id FROM goods;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN goods_cursor;
    
    read_loop: LOOP
        FETCH goods_cursor INTO goods_id_val;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- 为每个商品生成SKU
        INSERT INTO goods_skus (goods_id, sku_sn, spec_data, price, original_price, cost_price, stock, warn_stock, weight, volume, image_url, status, sales)
        SELECT 
            goods_id_val,
            CONCAT('SKU', LPAD(seq, 6, '0')),
            CASE 
                WHEN RAND() > 0.5 THEN 
                    JSON_OBJECT(
                        '1', (SELECT id FROM goods_spec_values WHERE spec_id = 1 ORDER BY RAND() LIMIT 1),
                        '2', (SELECT id FROM goods_spec_values WHERE spec_id = 2 ORDER BY RAND() LIMIT 1)
                    )
                ELSE 
                    JSON_OBJECT(
                        '1', (SELECT id FROM goods_spec_values WHERE spec_id = 1 ORDER BY RAND() LIMIT 1),
                        '2', (SELECT id FROM goods_spec_values WHERE spec_id = 2 ORDER BY RAND() LIMIT 1),
                        '3', (SELECT id FROM goods_spec_values WHERE spec_id = 3 ORDER BY RAND() LIMIT 1)
                    )
            END,
            FLOOR(RAND() * 2000) + 500,
            FLOOR(RAND() * 2500) + 600,
            FLOOR(RAND() * 1500) + 400,
            FLOOR(RAND() * 300) + 50,
            20,
            CASE 
                WHEN RAND() > 0.7 THEN 0.5
                WHEN RAND() > 0.4 THEN 1.0
                ELSE 0.2
            END,
            CASE 
                WHEN RAND() > 0.7 THEN 0.01
                WHEN RAND() > 0.4 THEN 0.005
                ELSE 0.001
            END,
            CONCAT('/skus/', goods_id_val, '-', seq, '.jpg'),
            1,
            FLOOR(RAND() * 100)
        FROM (
            SELECT a.N + b.N * 10 + 1 AS seq
            FROM (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) a
            CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) b
            LIMIT skus_per_goods
        ) seq_table;
        
        SET goods_count = goods_count + 1;
        
        -- 每处理100个商品输出一次进度
        IF goods_count % 100 = 0 THEN
            SELECT CONCAT('Generated SKUs for ', goods_count, ' goods, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END LOOP;
    
    CLOSE goods_cursor;
    
    SELECT CONCAT('Total generated SKUs for ', goods_count, ' goods, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 四、批量订单数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_orders(IN num_orders INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE batch_size INT DEFAULT 1000;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    DECLARE user_count INT;
    DECLARE goods_count INT;
    
    SELECT COUNT(*) INTO user_count FROM users;
    SELECT COUNT(*) INTO goods_count FROM goods;
    
    WHILE i <= num_orders DO
        -- 批量插入订单数据
        INSERT INTO orders (order_sn, user_id, order_status, shipping_status, pay_status, consignee, country, province, city, district, address, mobile, postscript, shipping_fee, discount_fee, coupon_fee, integral_fee, order_amount, total_amount, tax_fee, service_fee, platform_discount, pay_time, shipping_time, confirm_time, transaction_id, shipping_company, shipping_no, source, is_deleted, created_at)
        SELECT 
            CONCAT('ORDER', DATE_FORMAT(create_date, '%Y%m%d'), LPAD(seq, 8, '0')),
            (SELECT id FROM users ORDER BY RAND() LIMIT 1),
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 7 DAY) THEN 3 -- 已完成
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 3 DAY) THEN 2 -- 待收货
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 1 DAY) THEN 1 -- 待发货
                ELSE 0 -- 待付款
            END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 5 DAY) THEN 2 -- 已收货
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 2 DAY) THEN 1 -- 已发货
                ELSE 0 -- 未发货
            END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 1 DAY) THEN 1 -- 已支付
                ELSE 0 -- 未支付
            END,
            CONCAT('收货人', seq),
            '中国',
            CASE FLOOR(RAND() * 5)
                WHEN 0 THEN '北京市'
                WHEN 1 THEN '上海市'
                WHEN 2 THEN '广东省'
                WHEN 3 THEN '浙江省'
                ELSE '江苏省'
            END,
            CASE 
                WHEN province = '北京市' THEN '北京市'
                WHEN province = '上海市' THEN '上海市'
                ELSE CASE FLOOR(RAND() * 3)
                    WHEN 0 THEN '广州市'
                    WHEN 1 THEN '深圳市'
                    WHEN 2 THEN '杭州市'
                    WHEN 3 THEN '南京市'
                END
            END,
            CASE FLOOR(RAND() * 5)
                WHEN 0 THEN '朝阳区'
                WHEN 1 THEN '浦东新区'
                WHEN 2 THEN '天河区'
                WHEN 3 THEN '西湖区'
                ELSE '鼓楼区'
            END,
            CONCAT(province, city, district, '街道', FLOOR(RAND() * 100), '号'),
            CONCAT('13', FLOOR(5 + RAND() * 4), LPAD(FLOOR(RAND() * 100000000), 8, '0')),
            CASE WHEN RAND() > 0.8 THEN CONCAT('订单备注', seq) ELSE NULL END,
            CASE 
                WHEN RAND() > 0.7 THEN 15.00
                WHEN RAND() > 0.5 THEN 10.00
                ELSE 0.00
            END,
            CASE WHEN RAND() > 0.7 THEN FLOOR(RAND() * 50) + 10 ELSE 0 END,
            CASE WHEN RAND() > 0.8 THEN FLOOR(RAND() * 30) + 5 ELSE 0 END,
            CASE WHEN RAND() > 0.9 THEN FLOOR(RAND() * 20) + 5 ELSE 0 END,
            total_amount - discount_fee - coupon_fee - integral_fee + shipping_fee,
            total_amount,
            CASE WHEN RAND() > 0.9 THEN FLOOR(RAND() * 20) + 5 ELSE 0 END,
            CASE WHEN RAND() > 0.9 THEN FLOOR(RAND() * 10) + 5 ELSE 0 END,
            CASE WHEN RAND() > 0.8 THEN FLOOR(RAND() * 30) + 10 ELSE 0 END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 1 DAY) THEN DATE_ADD(create_date, INTERVAL FLOOR(RAND() * 60) MINUTE)
                ELSE NULL
            END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 2 DAY) THEN DATE_ADD(create_date, INTERVAL FLOOR(RAND() * 24 + 1) HOUR)
                ELSE NULL
            END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 5 DAY) THEN DATE_ADD(create_date, INTERVAL FLOOR(RAND() * 72 + 24) HOUR)
                ELSE NULL
            END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 1 DAY) THEN CONCAT('TRANS', LPAD(FLOOR(RAND() * 1000000000), 9, '0'))
                ELSE NULL
            END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 2 DAY) THEN 
                    CASE FLOOR(RAND() * 4)
                        WHEN 0 THEN '顺丰速运'
                        WHEN 1 THEN '中通快递'
                        WHEN 2 THEN '圆通速递'
                        ELSE '韵达快递'
                    END
                ELSE NULL
            END,
            CASE 
                WHEN create_date < DATE_SUB(NOW(), INTERVAL 2 DAY) THEN CONCAT('SF', LPAD(FLOOR(RAND() * 100000000), 8, '0'))
                ELSE NULL
            END,
            FLOOR(RAND() * 4) + 1,
            0,
            create_date
        FROM (
            SELECT 
                seq,
                CASE 
                    WHEN RAND() > 0.5 THEN DATE_ADD('2023-01-01', INTERVAL FLOOR(RAND() * 365) DAY)
                    ELSE DATE_ADD('2024-01-01', INTERVAL FLOOR(RAND() * DAYOFYEAR(CURDATE())) DAY)
                END AS create_date,
                FLOOR(RAND() * 500) + 100 AS total_amount
            FROM (
                SELECT a.N + b.N * 10 + c.N * 100 + d.N * 1000 + i AS seq
                FROM (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) a
                CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) b
                CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) c
                CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) d
                LIMIT batch_size
            ) seq_table
        ) order_data;
        
        SET i = i + batch_size;
        
        -- 每处理10000条输出一次进度
        IF i % 10000 = 1 THEN
            SELECT CONCAT('Generated ', i-1, ' orders, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END WHILE;
    
    SELECT CONCAT('Total generated ', num_orders, ' orders, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 五、批量订单商品数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_order_items(IN items_per_order INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE order_id_val BIGINT;
    DECLARE order_count INT DEFAULT 0;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    
    -- 声明游标
    DECLARE order_cursor CURSOR FOR SELECT id FROM orders;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN order_cursor;
    
    read_loop: LOOP
        FETCH order_cursor INTO order_id_val;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- 为每个订单生成商品项
        INSERT INTO order_items (order_id, goods_id, goods_name, goods_sn, sku_id, sku_sn, spec_data, goods_number, market_price, goods_price, goods_attr, is_real, is_gift)
        SELECT 
            order_id_val,
            g.id,
            g.goods_name,
            g.goods_sn,
            gs.id,
            gs.sku_sn,
            gs.spec_data,
            FLOOR(RAND() * 3) + 1,
            g.market_price,
            gs.price,
            CASE 
                WHEN RAND() > 0.5 THEN '颜色:黑色;内存:8GB;存储:128GB'
                ELSE '颜色:银色;内存:16GB;存储:256GB'
            END,
            1,
            CASE WHEN RAND() > 0.9 THEN 1 ELSE 0 END
        FROM goods g
        JOIN goods_skus gs ON g.id = gs.goods_id
        WHERE g.is_on_sale = 1
        ORDER BY RAND()
        LIMIT items_per_order;
        
        SET order_count = order_count + 1;
        
        -- 每处理1000个订单输出一次进度
        IF order_count % 1000 = 0 THEN
            SELECT CONCAT('Generated items for ', order_count, ' orders, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END LOOP;
    
    CLOSE order_cursor;
    
    SELECT CONCAT('Total generated items for ', order_count, ' orders, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 六、批量购物车数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_cart_items(IN carts_per_user INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE user_id_val BIGINT;
    DECLARE user_count INT DEFAULT 0;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    
    -- 声明游标
    DECLARE user_cursor CURSOR FOR SELECT id FROM users;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN user_cursor;
    
    read_loop: LOOP
        FETCH user_cursor INTO user_id_val;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- 为每个用户生成购物车项
        INSERT INTO cart (user_id, session_id, goods_id, sku_id, goods_name, quantity, price, market_price, spec_data, selected, created_at)
        SELECT 
            user_id_val,
            NULL,
            g.id,
            gs.id,
            g.goods_name,
            FLOOR(RAND() * 3) + 1,
            gs.price,
            g.market_price,
            gs.spec_data,
            1,
            DATE_ADD(NOW(), INTERVAL -FLOOR(RAND() * 30) DAY)
        FROM goods g
        JOIN goods_skus gs ON g.id = gs.goods_id
        WHERE g.is_on_sale = 1
        ORDER BY RAND()
        LIMIT carts_per_user;
        
        SET user_count = user_count + 1;
        
        -- 每处理1000个用户输出一次进度
        IF user_count % 1000 = 0 THEN
            SELECT CONCAT('Generated cart items for ', user_count, ' users, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END LOOP;
    
    CLOSE user_cursor;
    
    SELECT CONCAT('Total generated cart items for ', user_count, ' users, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 七、批量评价数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_comments(IN comments_per_goods INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE goods_id_val BIGINT;
    DECLARE goods_count INT DEFAULT 0;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    
    -- 声明游标
    DECLARE goods_cursor CURSOR FOR SELECT id FROM goods WHERE is_on_sale = 1;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN goods_cursor;
    
    read_loop: LOOP
        FETCH goods_cursor INTO goods_id_val;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- 为每个商品生成评价
        INSERT INTO goods_comments (order_id, goods_id, user_id, username, content, comment_rank, is_anonymous, has_picture, reply_content, reply_time, created_at)
        SELECT 
            o.id,
            goods_id_val,
            o.user_id,
            u.nickname,
            CASE FLOOR(RAND() * 5)
                WHEN 0 THEN '商品质量很好，非常满意'
                WHEN 1 THEN '物流很快，包装完好'
                WHEN 2 THEN '与描述相符，性价比高'
                WHEN 3 THEN '一般般，没有想象中好'
                WHEN 4 THEN '不太满意，有待改进'
            END,
            FLOOR(RAND() * 5) + 1,
            CASE WHEN RAND() > 0.8 THEN 1 ELSE 0 END,
            CASE WHEN RAND() > 0.7 THEN 1 ELSE 0 END,
            CASE WHEN RAND() > 0.5 THEN '感谢您的评价，我们会继续努力！' ELSE NULL END,
            CASE WHEN RAND() > 0.5 THEN DATE_ADD(o.created_at, INTERVAL FLOOR(RAND() * 48) + 24 HOUR) ELSE NULL END,
            DATE_ADD(o.created_at, INTERVAL FLOOR(RAND() * 24) + 1 HOUR)
        FROM orders o
        JOIN users u ON o.user_id = u.id
        JOIN order_items oi ON o.id = oi.order_id AND oi.goods_id = goods_id_val
        WHERE o.order_status = 3
        ORDER BY RAND()
        LIMIT comments_per_goods;
        
        SET goods_count = goods_count + 1;
        
        -- 每处理500个商品输出一次进度
        IF goods_count % 500 = 0 THEN
            SELECT CONCAT('Generated comments for ', goods_count, ' goods, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END LOOP;
    
    CLOSE goods_cursor;
    
    SELECT CONCAT('Total generated comments for ', goods_count, ' goods, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 八、批量用户行为数据生成存储过程

```sql
DELIMITER //

CREATE PROCEDURE generate_user_behaviors(IN behaviors_per_user INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE user_id_val BIGINT;
    DECLARE user_count INT DEFAULT 0;
    DECLARE start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    DECLARE goods_count INT;
    
    SELECT COUNT(*) INTO goods_count FROM goods;
    
    -- 声明游标
    DECLARE user_cursor CURSOR FOR SELECT id FROM users;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN user_cursor;
    
    read_loop: LOOP
        FETCH user_cursor INTO user_id_val;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- 为每个用户生成行为数据
        INSERT INTO user_behavior_log (user_id, behavior_type, page_url, goods_id, sku_id, stay_duration, device_info, ip_address, location, created_at)
        SELECT 
            user_id_val,
            CASE FLOOR(RAND() * 5)
                WHEN 0 THEN 1 -- 浏览
                WHEN 1 THEN 2 -- 点击
                WHEN 2 THEN 3 -- 收藏
                WHEN 3 THEN 4 -- 加购
                ELSE 5 -- 下单
            END,
            CONCAT('/goods/', g.id),
            g.id,
            (SELECT id FROM goods_skus WHERE goods_id = g.id ORDER BY RAND() LIMIT 1),
            FLOOR(RAND() * 60) + 10,
            CASE FLOOR(RAND() * 3)
                WHEN 0 THEN 'iPhone'
                WHEN 1 THEN 'Android'
                ELSE 'PC'
            END,
            CONCAT('192.168.', FLOOR(RAND() * 255), '.', FLOOR(RAND() * 255)),
            CASE FLOOR(RAND() * 5)
                WHEN 0 THEN '北京市'
                WHEN 1 THEN '上海市'
                WHEN 2 THEN '广州市'
                WHEN 3 THEN '深圳市'
                ELSE '杭州市'
            END,
            DATE_ADD(NOW(), INTERVAL -FLOOR(RAND() * 30) DAY)
        FROM goods g
        WHERE g.is_on_sale = 1
        ORDER BY RAND()
        LIMIT behaviors_per_user;
        
        SET user_count = user_count + 1;
        
        -- 每处理1000个用户输出一次进度
        IF user_count % 1000 = 0 THEN
            SELECT CONCAT('Generated behaviors for ', user_count, ' users, elapsed time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS progress;
        END IF;
    END LOOP;
    
    CLOSE user_cursor;
    
    SELECT CONCAT('Total generated behaviors for ', user_count, ' users, total time: ', TIMESTAMPDIFF(SECOND, start_time, CURRENT_TIMESTAMP), ' seconds') AS summary;
END //

DELIMITER ;
```

## 使用说明

1. **执行顺序**：
   - 先执行基础数据填充（用户、商品分类、品牌等）
   - 然后按顺序执行：用户→商品→SKU→订单→订单商品→购物车→评价→行为

2. **调用示例**：
```sql
-- 生成100万用户
CALL generate_users(1000000);

-- 生成10万商品
CALL generate_goods(100000);

-- 为每个商品生成3-5个SKU
CALL generate_skus(3);

-- 生成100万订单
CALL generate_orders(1000000);

-- 为每个订单生成1-5个商品项
CALL generate_order_items(3);

-- 为每个用户生成1-3个购物车项
CALL generate_cart_items(2);

-- 为每个商品生成5-10条评价
CALL generate_comments(8);

-- 为每个用户生成10-20条行为数据
CALL generate_user_behaviors(15);
```

3. **性能优化建议**：
   - 在非高峰期执行
   - 分批执行，避免一次性生成过多数据
   - 考虑临时关闭外键约束检查（SET FOREIGN_KEY_CHECKS=0）
   - 执行前关闭自动提交（SET autocommit=0）

4. **注意事项**：
   - 数据量越大，执行时间越长
   - 建议先在测试环境执行
   - 根据服务器配置调整批量大小