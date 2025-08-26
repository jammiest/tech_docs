# 电商数据库常用查询操作完整版

## 一、商品模块查询

### 1. 商品基础信息查询
```sql
-- 查询单个商品详情
SELECT * FROM goods WHERE id = ?;

-- 查询商品列表（分页+排序）
SELECT 
    g.id, g.goods_name, g.main_image, g.shop_price, 
    MIN(gs.price) AS min_price, g.sales_count, g.comment_count,
    ROUND(AVG(gc.rating),1) AS avg_rating
FROM goods g
LEFT JOIN goods_skus gs ON g.id = gs.goods_id
LEFT JOIN goods_comments gc ON g.id = gc.goods_id
WHERE g.is_on_sale = 1
GROUP BY g.id
ORDER BY 
    CASE WHEN ? = 'price_asc' THEN min_price END ASC,
    CASE WHEN ? = 'sales_desc' THEN g.sales_count END DESC,
    CASE WHEN ? = 'rating_desc' THEN avg_rating END DESC
LIMIT ?, ?;

-- 查询商品分类树
WITH RECURSIVE category_tree AS (
    SELECT id, parent_id, category_name, category_level, sort_order
    FROM goods_categories
    WHERE parent_id = 0
    
    UNION ALL
    
    SELECT c.id, c.parent_id, c.category_name, c.category_level, c.sort_order
    FROM goods_categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree
ORDER BY sort_order;
```

### 2. 商品搜索与筛选
```sql
-- 多条件商品搜索（支持分类、价格区间、关键词）
SELECT 
    g.id, g.goods_name, g.main_image,
    MIN(gs.price) AS min_price, MAX(gs.price) AS max_price,
    g.sales_count, g.comment_count,
    ROUND(AVG(gc.rating),1) AS avg_rating
FROM goods g
JOIN goods_skus gs ON g.id = gs.goods_id
LEFT JOIN goods_comments gc ON g.id = gc.goods_id
WHERE g.is_on_sale = 1
AND (? IS NULL OR g.category_id = ?)
AND (? IS NULL OR g.brand_id = ?)
AND gs.price BETWEEN ? AND ?
AND (g.goods_name LIKE CONCAT('%',?,'%') OR g.keywords LIKE CONCAT('%',?,'%'))
GROUP BY g.id
HAVING (? IS NULL OR avg_rating >= ?)
ORDER BY 
    CASE WHEN ? = 'price_asc' THEN min_price END ASC,
    CASE WHEN ? = 'price_desc' THEN max_price END DESC
LIMIT ?, ?;

-- 商品属性筛选
SELECT DISTINCT g.id
FROM goods g
JOIN goods_attr_relation gar ON g.id = gar.goods_id
JOIN goods_attributes ga ON gar.attr_id = ga.id
WHERE ga.attr_name = ? 
AND gar.attr_value = ?
AND g.is_on_sale = 1;
```

### 3. 商品SKU查询
```sql
-- 查询商品所有SKU
SELECT 
    gs.id, gs.sku_sn, gs.spec_data, gs.price, gs.stock,
    gs.image_url, gs.status,
    (
        SELECT GROUP_CONCAT(CONCAT(s.spec_name,':',sv.value_name) SEPARATOR '; ')
        FROM JSON_TABLE(
            JSON_KEYS(gs.spec_data),
            '$[*]' COLUMNS (spec_id INT PATH '$')
        ) AS jt
        JOIN goods_specs s ON jt.spec_id = s.id
        JOIN goods_spec_values sv ON JSON_UNQUOTE(JSON_EXTRACT(gs.spec_data, CONCAT('$."', s.id, '"'))) = sv.id
    ) AS spec_text
FROM goods_skus gs
WHERE gs.goods_id = ?;

-- 根据规格组合查询特定SKU
SELECT *
FROM goods_skus
WHERE goods_id = ?
AND JSON_CONTAINS(spec_data, ?);
```

## 二、用户模块查询

### 1. 用户基础信息
```sql
-- 用户登录验证
SELECT id, username, password_hash, salt, status 
FROM users 
WHERE username = ? OR email = ? OR phone = ?;

-- 查询用户信息（带统计）
SELECT 
    u.*,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.id) AS order_count,
    (SELECT SUM(order_amount) FROM orders WHERE user_id = u.id) AS total_spend,
    (SELECT COUNT(*) FROM user_addresses WHERE user_id = u.id) AS address_count
FROM users u
WHERE u.id = ?;
```

### 2. 用户地址管理
```sql
-- 查询用户收货地址
SELECT * FROM user_addresses 
WHERE user_id = ? 
ORDER BY is_default DESC, id DESC;

-- 查询默认地址
SELECT * FROM user_addresses
WHERE user_id = ? AND is_default = 1
LIMIT 1;
```

### 3. 用户行为分析
```sql
-- 用户浏览历史
SELECT 
    g.id, g.goods_name, g.main_image, g.shop_price,
    bh.view_time
FROM browse_history bh
JOIN goods g ON bh.goods_id = g.id
WHERE bh.user_id = ?
ORDER BY bh.view_time DESC
LIMIT 10;

-- 用户收藏商品
SELECT 
    g.id, g.goods_name, g.main_image, g.shop_price,
    uf.created_at
FROM user_favorites uf
JOIN goods g ON uf.goods_id = g.id
WHERE uf.user_id = ?
ORDER BY uf.created_at DESC;
```

## 三、订单模块查询

### 1. 订单基础查询
```sql
-- 查询用户订单列表（分页）
SELECT 
    o.id, o.order_sn, o.order_status, o.order_amount,
    o.created_at, o.pay_time,
    (SELECT COUNT(*) FROM order_items WHERE order_id = o.id) AS item_count,
    (SELECT main_image FROM order_items WHERE order_id = o.id LIMIT 1) AS cover_image
FROM orders o
WHERE o.user_id = ?
ORDER BY o.id DESC
LIMIT ?, ?;

-- 订单详情查询
SELECT 
    o.*,
    (
        SELECT JSON_ARRAYAGG(
            JSON_OBJECT(
                'goods_id', oi.goods_id,
                'goods_name', oi.goods_name,
                'price', oi.goods_price,
                'quantity', oi.goods_number,
                'image', (SELECT main_image FROM goods WHERE id = oi.goods_id)
            )
        )
        FROM order_items oi 
        WHERE oi.order_id = o.id
    ) AS items,
    (
        SELECT JSON_ARRAYAGG(
            JSON_OBJECT(
                'action_time', oa.log_time,
                'action_status', oa.order_status,
                'action_note', oa.action_note
            )
        )
        FROM order_actions oa
        WHERE oa.order_id = o.id
        ORDER BY oa.log_time
    ) AS action_logs
FROM orders o
WHERE o.id = ?;
```

### 2. 订单状态统计
```sql
-- 订单状态统计
SELECT 
    order_status,
    COUNT(*) AS count,
    SUM(order_amount) AS total_amount
FROM orders
WHERE user_id = ?
GROUP BY order_status;

-- 订单物流查询
SELECT 
    o.order_sn, l.logistics_no, l.company_name,
    ls.status, ls.status_desc, ls.update_time
FROM orders o
JOIN logistics l ON o.id = l.order_id
JOIN logistics_status ls ON l.logistics_no = ls.logistics_no
WHERE o.id = ?
ORDER BY ls.update_time DESC;
```

## 四、购物车模块查询

### 1. 购物车查询
```sql
-- 查询用户购物车
SELECT 
    c.id, c.goods_id, g.goods_name, g.main_image,
    c.sku_id, gs.spec_data, c.quantity, 
    gs.price AS current_price, gs.stock AS current_stock,
    (gs.price * c.quantity) AS subtotal
FROM cart c
JOIN goods g ON c.goods_id = g.id
LEFT JOIN goods_skus gs ON c.sku_id = gs.id
WHERE c.user_id = ? AND c.selected = 1;

-- 购物车商品数量统计
SELECT 
    COUNT(*) AS total_count, 
    SUM(quantity) AS total_quantity,
    SUM(price * quantity) AS total_amount
FROM cart
WHERE user_id = ? AND selected = 1;
```

## 五、支付模块查询

### 1. 支付记录查询
```sql
-- 查询订单支付记录
SELECT * FROM payment_records
WHERE order_id = ?
ORDER BY id DESC;

-- 支付状态检查
SELECT 
    o.order_sn, p.payment_status, p.payment_amount,
    p.payment_time, p.transaction_id
FROM orders o
JOIN payment_records p ON o.id = p.order_id
WHERE o.id = ?;
```

## 六、数据分析查询

### 1. 销售数据分析
```sql
-- 商品销售排行
SELECT 
    g.id, g.goods_name, g.main_image,
    SUM(oi.goods_number) AS sales_volume,
    SUM(oi.goods_number * oi.goods_price) AS sales_amount
FROM order_items oi
JOIN goods g ON oi.goods_id = g.id
WHERE oi.created_at BETWEEN ? AND ?
GROUP BY g.id
ORDER BY sales_volume DESC
LIMIT 10;

-- 每日销售统计
SELECT 
    DATE(created_at) AS date,
    COUNT(*) AS order_count,
    SUM(order_amount) AS total_amount
FROM orders
WHERE created_at BETWEEN ? AND ?
GROUP BY DATE(created_at);
```

### 2. 用户行为分析
```sql
-- 用户购买转化率
SELECT 
    COUNT(DISTINCT v.user_id) AS view_users,
    COUNT(DISTINCT c.user_id) AS cart_users,
    COUNT(DISTINCT o.user_id) AS order_users,
    ROUND(COUNT(DISTINCT c.user_id)/COUNT(DISTINCT v.user_id)*100, 2) AS view_to_cart_rate,
    ROUND(COUNT(DISTINCT o.user_id)/COUNT(DISTINCT c.user_id)*100, 2) AS cart_to_order_rate
FROM user_behavior_log v
LEFT JOIN user_behavior_log c ON v.user_id = c.user_id AND c.behavior_type = 4 -- 加购
LEFT JOIN orders o ON v.user_id = o.user_id
WHERE v.behavior_type = 1 -- 浏览
AND v.created_at BETWEEN ? AND ?;
```

## 七、营销活动查询

### 1. 秒杀活动查询
```sql
-- 查询进行中的秒杀活动
SELECT 
    sa.id, sa.name, sa.start_time, sa.end_time,
    COUNT(sg.id) AS goods_count
FROM seckill_activity sa
LEFT JOIN seckill_goods sg ON sa.id = sg.activity_id
WHERE sa.start_time <= NOW() AND sa.end_time >= NOW()
AND sa.status = 1
GROUP BY sa.id;

-- 查询秒杀商品详情
SELECT 
    sg.*, g.goods_name, g.main_image, g.shop_price AS original_price
FROM seckill_goods sg
JOIN goods g ON sg.goods_id = g.id
WHERE sg.activity_id = ?;
```

### 2. 优惠券查询
```sql
-- 查询用户可用优惠券
SELECT 
    c.*, uc.id AS user_coupon_id, uc.status,
    CASE 
        WHEN NOW() < c.start_time THEN '未开始'
        WHEN NOW() > c.end_time THEN '已过期'
        WHEN uc.status = 1 THEN '已使用'
        ELSE '可使用'
    END AS coupon_status
FROM coupons c
JOIN user_coupons uc ON c.id = uc.coupon_id
WHERE uc.user_id = ?
AND uc.status = 0
AND NOW() BETWEEN c.start_time AND c.end_time;
```

## 八、性能优化查询

### 1. 慢查询分析
```sql
-- 查询慢查询日志
SELECT 
    query_time, lock_time, rows_examined, sql_text
FROM mysql.slow_log
ORDER BY query_time DESC
LIMIT 10;

-- 查询未使用索引的查询
SELECT * FROM mysql.slow_log
WHERE query_time > 1
AND sql_text NOT LIKE '%USE INDEX%'
ORDER BY query_time DESC;
```

### 2. 索引使用分析
```sql
-- 查看表索引使用情况
SELECT 
    table_name, index_name, seq_in_index, column_name,
    cardinality, nullable, index_type
FROM information_schema.statistics
WHERE table_schema = DATABASE()
ORDER BY table_name, index_name, seq_in_index;

-- 分析查询执行计划
EXPLAIN 
SELECT * FROM orders 
WHERE user_id = ? 
ORDER BY id DESC 
LIMIT 10;
```

## 九、高级查询技巧

### 1. 使用窗口函数
```sql
-- 查询用户订单排名
SELECT 
    user_id, order_sn, order_amount,
    RANK() OVER (PARTITION BY user_id ORDER BY order_amount DESC) AS amount_rank,
    DENSE_RANK() OVER (PARTITION BY user_id ORDER BY DATE(created_at)) AS day_rank
FROM orders
WHERE created_at BETWEEN ? AND ?;

-- 查询商品销售趋势
SELECT 
    goods_id,
    DATE(created_at) AS sale_date,
    SUM(goods_number) AS daily_sales,
    SUM(SUM(goods_number)) OVER (PARTITION BY goods_id ORDER BY DATE(created_at)) AS running_total
FROM order_items
GROUP BY goods_id, DATE(created_at);
```

### 2. 使用CTE递归查询
```sql
-- 查询分类层级路径
WITH RECURSIVE category_path AS (
    SELECT id, parent_id, category_name, category_name AS path
    FROM goods_categories
    WHERE parent_id = 0
    
    UNION ALL
    
    SELECT c.id, c.parent_id, c.category_name, 
           CONCAT(cp.path, ' > ', c.category_name)
    FROM goods_categories c
    JOIN category_path cp ON c.parent_id = cp.id
)
SELECT * FROM category_path
ORDER BY path;
```

### 3. JSON数据处理
```sql
-- 解析SKU规格数据
SELECT 
    gs.id,
    JSON_UNQUOTE(JSON_EXTRACT(gs.spec_data, '$.1')) AS color_id,
    JSON_UNQUOTE(JSON_EXTRACT(gs.spec_data, '$.2')) AS memory_id,
    JSON_UNQUOTE(JSON_EXTRACT(gs.spec_data, '$.3')) AS storage_id
FROM goods_skus gs
WHERE gs.goods_id = ?;

-- 构建JSON结果
SELECT 
    g.id,
    g.goods_name,
    (
        SELECT JSON_ARRAYAGG(
            JSON_OBJECT(
                'sku_id', gs.id,
                'price', gs.price,
                'spec_data', gs.spec_data
            )
        )
        FROM goods_skus gs
        WHERE gs.goods_id = g.id
    ) AS skus
FROM goods g
WHERE g.id = ?;
```

## 十、数据维护查询

### 1. 数据一致性检查
```sql
-- 检查订单金额一致性
SELECT 
    o.id, o.order_sn, o.order_amount,
    SUM(oi.goods_number * oi.goods_price) AS calculated_amount,
    o.order_amount - SUM(oi.goods_number * oi.goods_price) AS difference
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
GROUP BY o.id
HAVING ABS(difference) > 0.01;

-- 检查库存一致性
SELECT 
    gs.id, gs.stock AS db_stock,
    (SELECT COALESCE(SUM(stock_change), 0) FROM stock_log WHERE sku_id = gs.id) AS log_stock,
    gs.stock - (SELECT COALESCE(SUM(stock_change), 0) FROM stock_log WHERE sku_id = gs.id) AS difference
FROM goods_skus gs
HAVING ABS(difference) > 0;
```

### 2. 数据归档
```sql
-- 创建订单归档表
CREATE TABLE orders_archive LIKE orders;

-- 归档历史订单数据
INSERT INTO orders_archive
SELECT * FROM orders 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- 删除已归档数据
DELETE FROM orders 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

## 使用说明

1. 根据实际业务需求替换查询中的参数（如?占位符）
2. 复杂查询建议添加适当的索引以提高性能
3. 大数据量查询建议使用分页（LIMIT）
4. 生产环境使用前应在测试环境验证查询性能
5. 敏感数据查询应添加适当的权限控制

这套查询涵盖了电商系统的主要业务场景，包括商品展示、用户管理、订单处理、数据分析等核心功能，可根据实际业务需求进行调整和扩展。