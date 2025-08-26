# 电商数据库高级查询操作完整版

## 一、复杂商品查询

### 1. 多维度商品搜索（支持SKU属性筛选）
```sql
SELECT 
    g.id, g.goods_name, g.main_image,
    MIN(gs.price) AS min_price,
    MAX(gs.price) AS max_price,
    SUM(gs.stock) AS total_stock,
    JSON_ARRAYAGG(DISTINCT gc.category_name) AS categories,
    (
        SELECT JSON_ARRAYAGG(JSON_OBJECT('attr_name', ga.attr_name, 'attr_value', gar.attr_value))
        FROM goods_attr_relation gar
        JOIN goods_attributes ga ON gar.attr_id = ga.id
        WHERE gar.goods_id = g.id
    ) AS attributes
FROM goods g
JOIN goods_skus gs ON g.id = gs.goods_id
JOIN goods_categories gc ON g.category_id = gc.id
WHERE g.is_on_sale = 1
  AND EXISTS (
      SELECT 1 FROM goods_skus gs2
      WHERE gs2.goods_id = g.id
        AND gs2.price BETWEEN ? AND ?
  )
  AND (
      SELECT COUNT(*) 
      FROM goods_attr_relation gar2
      JOIN goods_attributes ga2 ON gar2.attr_id = ga2.id
      WHERE gar2.goods_id = g.id
        AND ga2.attr_name = ?
        AND gar2.attr_value = ?
  ) > 0
GROUP BY g.id
HAVING total_stock > 0
ORDER BY 
    CASE WHEN ? = 'price_asc' THEN min_price END ASC,
    CASE WHEN ? = 'sales_desc' THEN g.sales_count END DESC
LIMIT ? OFFSET ?;
```

### 2. 商品SKU组合查询（JSON数据处理）
```sql
SELECT 
    g.id AS goods_id,
    g.goods_name,
    (
        SELECT JSON_OBJECT(
            'spec_id', gs.id,
            'spec_name', gs.spec_name,
            'values', (
                SELECT JSON_ARRAYAGG(
                    JSON_OBJECT(
                        'value_id', gsv.id,
                        'value_name', gsv.value_name,
                        'spec_image', gsv.spec_image
                    )
                )
                FROM goods_spec_values gsv
                WHERE gsv.spec_id = gs.id
            )
        )
        FROM goods_specs gs
        WHERE gs.id IN (
            SELECT JSON_UNQUOTE(JSON_EXTRACT(spec_data, CONCAT('$."', spec_id, '"')))
            FROM goods_skus
            WHERE goods_id = g.id
            LIMIT 1
        )
    ) AS spec_tree,
    (
        SELECT JSON_ARRAYAGG(
            JSON_OBJECT(
                'sku_id', gsk.id,
                'price', gsk.price,
                'stock', gsk.stock,
                'spec_data', gsk.spec_data,
                'spec_text', (
                    SELECT GROUP_CONCAT(
                        CONCAT(gs2.spec_name, ':', gsv2.value_name) 
                        SEPARATOR '; '
                    )
                    FROM JSON_TABLE(
                        JSON_KEYS(gsk.spec_data),
                        '$[*]' COLUMNS (spec_id INT PATH '$')
                    ) AS jt
                    JOIN goods_specs gs2 ON jt.spec_id = gs2.id
                    JOIN goods_spec_values gsv2 ON JSON_UNQUOTE(JSON_EXTRACT(gsk.spec_data, CONCAT('$."', gs2.id, '"'))) = gsv2.id
                )
            )
        )
        FROM goods_skus gsk
        WHERE gsk.goods_id = g.id
    ) AS sku_list
FROM goods g
WHERE g.id = ?;
```

## 二、高级订单分析

### 1. 订单漏斗分析（CTE递归查询）
```sql
WITH order_funnel AS (
    SELECT 
        DATE(created_at) AS day,
        COUNT(DISTINCT CASE WHEN order_status = 0 THEN user_id END) AS unpaid_users,
        COUNT(DISTINCT CASE WHEN order_status = 1 THEN user_id END) AS paid_users,
        COUNT(DISTINCT CASE WHEN order_status = 2 THEN user_id END) AS shipped_users,
        COUNT(DISTINCT CASE WHEN order_status = 3 THEN user_id END) AS completed_users
    FROM orders
    WHERE created_at BETWEEN ? AND ?
    GROUP BY DATE(created_at)
)
SELECT 
    day,
    unpaid_users,
    paid_users,
    shipped_users,
    completed_users,
    ROUND(paid_users * 100.0 / NULLIF(unpaid_users, 0), 2) AS payment_conversion_rate,
    ROUND(shipped_users * 100.0 / NULLIF(paid_users, 0), 2) AS shipping_rate,
    ROUND(completed_users * 100.0 / NULLIF(shipped_users, 0), 2) AS completion_rate
FROM order_funnel
ORDER BY day;
```

### 2. 订单商品关联购买分析
```sql
SELECT 
    g1.id AS goods_id_1,
    g1.goods_name AS goods_name_1,
    g2.id AS goods_id_2,
    g2.goods_name AS goods_name_2,
    COUNT(DISTINCT o.user_id) AS co_purchase_count,
    ROUND(COUNT(DISTINCT o.user_id) * 100.0 / (
        SELECT COUNT(DISTINCT user_id) 
        FROM orders 
        WHERE created_at BETWEEN ? AND ?
    ), 4) AS co_purchase_rate
FROM orders o
JOIN order_items oi1 ON o.id = oi1.order_id
JOIN order_items oi2 ON o.id = oi2.order_id AND oi1.goods_id < oi2.goods_id
JOIN goods g1 ON oi1.goods_id = g1.id
JOIN goods g2 ON oi2.goods_id = g2.id
WHERE o.created_at BETWEEN ? AND ?
GROUP BY g1.id, g2.id
HAVING co_purchase_count > ?
ORDER BY co_purchase_count DESC
LIMIT 50;
```

## 三、用户行为分析

### 1. 用户路径分析（使用窗口函数）
```sql
WITH user_events_ranked AS (
    SELECT 
        user_id,
        event_type,
        event_time,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_time) AS event_seq
    FROM user_events
    WHERE user_id IN (
        SELECT DISTINCT user_id 
        FROM orders 
        WHERE created_at BETWEEN ? AND ?
    )
    AND event_time BETWEEN ? AND ?
)
SELECT 
    prev.event_type AS from_event,
    curr.event_type AS to_event,
    COUNT(*) AS transition_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY prev.event_type), 2) AS transition_rate
FROM user_events_ranked curr
JOIN user_events_ranked prev ON curr.user_id = prev.user_id 
    AND curr.event_seq = prev.event_seq + 1
GROUP BY prev.event_type, curr.event_type
ORDER BY transition_count DESC;
```

### 2. RFM用户分群分析
```sql
WITH user_stats AS (
    SELECT 
        u.id AS user_id,
        u.username,
        MAX(o.created_at) AS last_order_date,
        COUNT(o.id) AS order_count,
        SUM(o.order_amount) AS total_spend,
        DATEDIFF(NOW(), MAX(o.created_at)) AS recency_days
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE o.order_status = 3 -- 已完成订单
    GROUP BY u.id
),
rfm_scores AS (
    SELECT 
        user_id,
        username,
        last_order_date,
        order_count,
        total_spend,
        recency_days,
        NTILE(5) OVER (ORDER BY recency_days DESC) AS r_score,
        NTILE(5) OVER (ORDER BY order_count) AS f_score,
        NTILE(5) OVER (ORDER BY total_spend) AS m_score
    FROM user_stats
)
SELECT 
    CONCAT(r_score, f_score, m_score) AS rfm_segment,
    COUNT(*) AS user_count,
    AVG(recency_days) AS avg_recency,
    AVG(order_count) AS avg_frequency,
    AVG(total_spend) AS avg_monetary,
    CASE 
        WHEN CONCAT(r_score, f_score, m_score) IN ('555','554','545','544') THEN '高价值客户'
        WHEN CONCAT(r_score, f_score, m_score) LIKE '5__' THEN '新客户'
        WHEN CONCAT(r_score, f_score, m_score) LIKE '__5' THEN '高消费客户'
        WHEN CONCAT(r_score, f_score, m_score) LIKE '_5_' THEN '活跃客户'
        WHEN r_score = 1 THEN '流失风险客户'
        ELSE '一般客户'
    END AS segment_name
FROM rfm_scores
GROUP BY rfm_segment
ORDER BY user_count DESC;
```

## 四、库存与供应链分析

### 1. 库存周转率分析
```sql
SELECT 
    g.id AS goods_id,
    g.goods_name,
    gs.id AS sku_id,
    gs.spec_data,
    gs.stock AS current_stock,
    COALESCE(SUM(oi.goods_number), 0) AS sold_count,
    COALESCE(SUM(oi.goods_number * oi.goods_price), 0) AS sold_amount,
    CASE 
        WHEN COALESCE(SUM(oi.goods_number), 0) = 0 THEN NULL
        ELSE gs.stock / NULLIF(SUM(oi.goods_number), 0)
    END AS stock_turnover_days,
    CASE 
        WHEN gs.stock = 0 THEN NULL
        ELSE COALESCE(SUM(oi.goods_number), 0) * 30 / gs.stock
    END AS turnover_rate
FROM goods_skus gs
JOIN goods g ON gs.goods_id = g.id
LEFT JOIN order_items oi ON gs.id = oi.sku_id
    AND oi.created_at BETWEEN ? AND ?
GROUP BY g.id, gs.id
HAVING sold_count > 0
ORDER BY turnover_rate DESC
LIMIT 100;
```

### 2. 供应链时效分析
```sql
SELECT 
    l.shipping_company,
    COUNT(*) AS order_count,
    AVG(TIMESTAMPDIFF(HOUR, o.shipping_time, ls.update_time)) AS avg_shipping_hours,
    AVG(TIMESTAMPDIFF(HOUR, ls.update_time, o.confirm_time)) AS avg_delivery_hours,
    AVG(TIMESTAMPDIFF(HOUR, o.shipping_time, o.confirm_time)) AS avg_total_hours,
    SUM(CASE WHEN TIMESTAMPDIFF(HOUR, o.shipping_time, o.confirm_time) > 72 THEN 1 ELSE 0 END) AS delay_count
FROM orders o
JOIN logistics l ON o.id = l.order_id
JOIN logistics_status ls ON l.logistics_no = ls.logistics_no
    AND ls.status = '已发货'
WHERE o.shipping_time BETWEEN ? AND ?
    AND o.confirm_time IS NOT NULL
GROUP BY l.shipping_company
HAVING order_count > 10
ORDER BY avg_total_hours;
```

## 五、营销效果分析

### 1. 促销活动ROI分析
```sql
SELECT 
    p.id AS promotion_id,
    p.promotion_name,
    p.start_time,
    p.end_time,
    p.discount_amount AS promotion_cost,
    COUNT(DISTINCT o.id) AS order_count,
    SUM(o.order_amount) AS gross_sales,
    SUM(o.discount_fee) AS total_discount,
    SUM(oi.goods_number) AS item_count,
    COUNT(DISTINCT o.user_id) AS user_count,
    ROUND(SUM(o.order_amount) / NULLIF(p.discount_amount, 0), 2) AS roi
FROM promotions p
LEFT JOIN orders o ON o.promotion_id = p.id
    AND o.created_at BETWEEN p.start_time AND p.end_time
LEFT JOIN order_items oi ON o.id = oi.order_id
WHERE p.start_time BETWEEN ? AND ?
GROUP BY p.id
ORDER BY roi DESC;
```

### 2. 优惠券使用分析
```sql
SELECT 
    c.coupon_name,
    c.discount_type,
    c.discount_value,
    c.min_order_amount,
    COUNT(DISTINCT uc.id) AS issued_count,
    COUNT(DISTINCT CASE WHEN uc.used_time IS NOT NULL THEN uc.id END) AS used_count,
    COUNT(DISTINCT CASE WHEN uc.used_time IS NULL AND uc.expire_time > NOW() THEN uc.id END) AS valid_count,
    ROUND(COUNT(DISTINCT CASE WHEN uc.used_time IS NOT NULL THEN uc.id END) * 100.0 / 
          NULLIF(COUNT(DISTINCT uc.id), 0), 2) AS redemption_rate,
    SUM(CASE WHEN uc.used_time IS NOT NULL THEN o.order_amount ELSE 0 END) AS gross_sales,
    SUM(CASE WHEN uc.used_time IS NOT NULL THEN c.discount_value ELSE 0 END) AS total_discount
FROM coupons c
LEFT JOIN user_coupons uc ON c.id = uc.coupon_id
LEFT JOIN orders o ON uc.id = o.coupon_id
WHERE c.create_time BETWEEN ? AND ?
GROUP BY c.id
ORDER BY redemption_rate DESC;
```

## 六、性能优化技巧

### 1. 查询执行计划分析
```sql
-- 查看详细执行计划
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = ? ORDER BY id DESC LIMIT 10;

-- 索引使用情况检查
SELECT 
    table_name,
    index_name,
    GROUP_CONCAT(column_name ORDER BY seq_in_index) AS columns,
    index_type,
    CASE non_unique WHEN 0 THEN 'UNIQUE' ELSE 'NON_UNIQUE' END AS uniqueness
FROM information_schema.statistics
WHERE table_schema = DATABASE()
GROUP BY table_name, index_name;
```

### 2. 慢查询优化示例
```sql
-- 优化前（全表扫描）
SELECT * FROM orders WHERE DATE(created_at) = '2023-01-01';

-- 优化后（使用索引范围查询）
SELECT * FROM orders 
WHERE created_at >= '2023-01-01 00:00:00' 
  AND created_at < '2023-01-02 00:00:00';

-- 优化前（OR条件导致索引失效）
SELECT * FROM orders 
WHERE order_status = 1 OR pay_status = 1;

-- 优化后（使用UNION优化OR条件）
SELECT * FROM orders WHERE order_status = 1
UNION
SELECT * FROM orders WHERE pay_status = 1;
```

## 七、高级报表查询

### 1. 销售日报表（PIVOT效果）
```sql
SELECT 
    DATE(created_at) AS report_date,
    COUNT(*) AS order_count,
    SUM(order_amount) AS total_amount,
    SUM(CASE WHEN payment_method = 1 THEN order_amount ELSE 0 END) AS alipay_amount,
    SUM(CASE WHEN payment_method = 2 THEN order_amount ELSE 0 END) AS wechat_amount,
    SUM(CASE WHEN source = 1 THEN 1 ELSE 0 END) AS pc_orders,
    SUM(CASE WHEN source = 2 THEN 1 ELSE 0 END) AS app_orders,
    SUM(CASE WHEN source = 3 THEN 1 ELSE 0 END) AS mini_program_orders
FROM orders
WHERE created_at BETWEEN ? AND ?
GROUP BY DATE(created_at)
ORDER BY report_date;
```

### 2. 商品销售趋势分析（时间序列）
```sql
WITH date_series AS (
    SELECT DATE(?) + INTERVAL seq DAY AS report_date
    FROM (
        SELECT 0 AS seq UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4
        UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9
    ) AS seq
    WHERE DATE(?) + INTERVAL seq DAY <= DATE(?)
)
SELECT 
    ds.report_date,
    g.id AS goods_id,
    g.goods_name,
    COALESCE(SUM(oi.goods_number), 0) AS sales_volume,
    COALESCE(SUM(oi.goods_number * oi.goods_price), 0) AS sales_amount
FROM date_series ds
CROSS JOIN goods g
LEFT JOIN order_items oi ON g.id = oi.goods_id
    AND DATE(oi.created_at) = ds.report_date
    AND oi.created_at BETWEEN ? AND ?
WHERE g.id IN (?)
GROUP BY ds.report_date, g.id
ORDER BY ds.report_date, sales_amount DESC;
```

## 八、数据清洗与维护

### 1. 数据一致性检查
```sql
-- 订单金额一致性检查
SELECT 
    o.id AS order_id,
    o.order_sn,
    o.order_amount AS declared_amount,
    SUM(oi.goods_number * oi.goods_price) AS calculated_amount,
    o.order_amount - SUM(oi.goods_number * oi.goods_price) AS difference
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
GROUP BY o.id
HAVING ABS(difference) > 0.01
LIMIT 100;

-- 库存一致性检查
SELECT 
    gs.id AS sku_id,
    gs.stock AS db_stock,
    (SELECT COALESCE(SUM(stock_change), 0) FROM stock_log WHERE sku_id = gs.id) AS log_stock,
    gs.stock - (SELECT COALESCE(SUM(stock_change), 0) FROM stock_log WHERE sku_id = gs.id) AS difference
FROM goods_skus gs
HAVING ABS(difference) > 0
ORDER BY ABS(difference) DESC;
```

### 2. 历史数据归档
```sql
-- 创建归档表
CREATE TABLE orders_archive LIKE orders;

-- 归档历史数据
INSERT INTO orders_archive
SELECT * FROM orders 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- 删除已归档数据
DELETE FROM orders 
WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```