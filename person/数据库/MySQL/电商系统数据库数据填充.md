# 电商数据库数据填充脚本

## 一、基础数据填充

### 1. 用户数据填充
```sql
-- 插入管理员用户
INSERT INTO users (username, nickname, email, phone, password_hash, salt, gender, birthday, avatar, status, register_time, register_ip)
VALUES 
('admin', '系统管理员', 'admin@example.com', '13800000000', '5f4dcc3b5aa765d61d8327deb882cf99', 'salt123', 1, '1990-01-01', '/avatars/admin.jpg', 1, NOW(), '127.0.0.1'),
('testuser', '测试用户', 'test@example.com', '13900000001', '5f4dcc3b5aa765d61d8327deb882cf99', 'salt456', 0, '1995-05-15', '/avatars/user1.jpg', 1, NOW(), '192.168.1.1');

-- 批量插入普通用户
INSERT INTO users (username, nickname, email, phone, password_hash, salt, gender, birthday, avatar, status, register_time, register_ip)
SELECT 
  CONCAT('user', seq), 
  CONCAT('用户', seq),
  CONCAT('user', seq, '@example.com'),
  CONCAT('138', FLOOR(RAND() * 100000000)),
  '5f4dcc3b5aa765d61d8327deb882cf99',
  CONCAT('salt', FLOOR(RAND() * 1000)),
  FLOOR(RAND() * 3),
  DATE_ADD('1990-01-01', INTERVAL FLOOR(RAND() * 365*30) DAY),
  CONCAT('/avatars/user', seq, '.jpg'),
  1,
  DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 365*3) DAY),
  CONCAT('192.168.', FLOOR(RAND() * 255), '.', FLOOR(RAND() * 255))
FROM (
  SELECT a.N + b.N * 10 + c.N * 100 + 1 AS seq
  FROM (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) a
  CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) b
  CROSS JOIN (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) c
  LIMIT 100
) seq_table;
```

### 2. 用户地址数据填充
```sql
-- 为每个用户插入1-3个收货地址
INSERT INTO user_addresses (user_id, receiver_name, receiver_phone, province, city, district, detailed_address, postal_code, is_default)
SELECT 
  u.id,
  CASE 
    WHEN RAND() < 0.5 THEN u.nickname 
    ELSE CONCAT('收货人', FLOOR(RAND() * 100)) 
  END,
  CONCAT('13', FLOOR(5 + RAND() * 4), FLOOR(RAND() * 100000000)),
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
  CONCAT(FLOOR(RAND() * 8 + 1), FLOOR(RAND() * 100000)),
  CASE WHEN addr_num = 1 THEN 1 ELSE 0 END
FROM (
  SELECT id, nickname FROM users
) u
CROSS JOIN (
  SELECT 1 AS addr_num UNION SELECT 2 UNION SELECT 3
) addr_counts
WHERE RAND() < 0.7 OR addr_num = 1;
```

## 二、商品数据填充

### 1. 商品分类数据填充
```sql
-- 一级分类
INSERT INTO goods_categories (parent_id, category_name, category_level, sort_order, is_visible, icon_url, keywords, description)
VALUES 
(0, '手机数码', 1, 1, 1, '/icons/phone.png', '手机,数码,电子', '手机数码产品分类'),
(0, '电脑办公', 1, 2, 1, '/icons/computer.png', '电脑,笔记本,办公', '电脑办公产品分类'),
(0, '家用电器', 1, 3, 1, '/icons/appliance.png', '家电,电器,家居', '家用电器产品分类');

-- 二级分类
INSERT INTO goods_categories (parent_id, category_name, category_level, sort_order, is_visible, icon_url, keywords, description)
SELECT 
  id,
  CASE id
    WHEN 1 THEN '智能手机'
    WHEN 2 THEN '笔记本电脑'
    WHEN 3 THEN '大家电'
  END,
  2,
  1,
  1,
  '/icons/sub1.png',
  CASE id
    WHEN 1 THEN '智能手机,5G手机'
    WHEN 2 THEN '笔记本,电脑'
    WHEN 3 THEN '冰箱,空调'
  END,
  CONCAT(category_name, '子分类')
FROM goods_categories 
WHERE category_level = 1;

-- 三级分类
INSERT INTO goods_categories (parent_id, category_name, category_level, sort_order, is_visible, icon_url, keywords, description)
SELECT 
  id,
  CASE parent_id
    WHEN 1 THEN '5G手机'
    WHEN 2 THEN '游戏本'
    WHEN 3 THEN '智能冰箱'
  END,
  3,
  1,
  1,
  '/icons/sub2.png',
  CASE parent_id
    WHEN 1 THEN '5G,旗舰手机'
    WHEN 2 THEN '游戏,电竞'
    WHEN 3 THEN '智能,物联网'
  END,
  CONCAT(category_name, '细分分类')
FROM goods_categories 
WHERE category_level = 2;
```

### 2. 品牌数据填充
```sql
INSERT INTO brands (brand_name, brand_logo, brand_desc, sort_order, is_show)
VALUES 
('Apple', '/brands/apple.png', '苹果公司', 1, 1),
('Huawei', '/brands/huawei.png', '华为技术有限公司', 2, 1),
('Xiaomi', '/brands/xiaomi.png', '小米科技', 3, 1),
('Lenovo', '/brands/lenovo.png', '联想集团', 4, 1),
('Haier', '/brands/haier.png', '海尔集团', 5, 1);
```

### 3. 商品规格数据填充
```sql
-- 规格类型
INSERT INTO goods_specs (spec_name, spec_type, sort_order)
VALUES 
('颜色', 2, 1),
('内存', 1, 2),
('存储', 1, 3),
('尺寸', 1, 4);

-- 规格值
INSERT INTO goods_spec_values (spec_id, value_name, spec_image, sort_order)
VALUES 
-- 颜色规格
(1, '黑色', '/specs/color_black.png', 1),
(1, '白色', '/specs/color_white.png', 2),
(1, '蓝色', '/specs/color_blue.png', 3),
-- 内存规格
(2, '4GB', NULL, 1),
(2, '6GB', NULL, 2),
(2, '8GB', NULL, 3),
-- 存储规格
(3, '64GB', NULL, 1),
(3, '128GB', NULL, 2),
(3, '256GB', NULL, 3),
-- 尺寸规格
(4, '5.5英寸', NULL, 1),
(4, '6.1英寸', NULL, 2),
(4, '6.7英寸', NULL, 3);
```

### 4. 商品数据填充
```sql
-- 插入商品基础数据
INSERT INTO goods (category_id, goods_name, goods_sn, brand_id, market_price, shop_price, cost_price, stock, warn_stock, goods_weight, goods_brief, goods_desc, main_image, is_on_sale, is_recommend, is_new, is_hot, sales_count, comment_count, rating)
SELECT 
  c.id,
  CASE 
    WHEN c.category_name = '智能手机' THEN CONCAT(b.brand_name, ' ', FLOOR(RAND() * 10) + 10, ' Pro')
    WHEN c.category_name = '笔记本电脑' THEN CONCAT(b.brand_name, ' ', '笔记本 ', FLOOR(RAND() * 5) + 2020)
    WHEN c.category_name = '大家电' THEN CONCAT(b.brand_name, ' ', '智能', CASE FLOOR(RAND() * 3) WHEN 0 THEN '冰箱' WHEN 1 THEN '空调' ELSE '洗衣机' END)
  END,
  CONCAT('SN', DATE_FORMAT(NOW(), '%Y%m%d'), FLOOR(RAND() * 10000)),
  b.id,
  CASE 
    WHEN c.category_name = '智能手机' THEN FLOOR(RAND() * 3000) + 3000
    WHEN c.category_name = '笔记本电脑' THEN FLOOR(RAND() * 5000) + 5000
    WHEN c.category_name = '大家电' THEN FLOOR(RAND() * 4000) + 2000
  END,
  CASE 
    WHEN c.category_name = '智能手机' THEN FLOOR(RAND() * 2000) + 2000
    WHEN c.category_name = '笔记本电脑' THEN FLOOR(RAND() * 3000) + 3000
    WHEN c.category_name = '大家电' THEN FLOOR(RAND() * 2000) + 1000
  END,
  CASE 
    WHEN c.category_name = '智能手机' THEN FLOOR(RAND() * 1500) + 1500
    WHEN c.category_name = '笔记本电脑' THEN FLOOR(RAND() * 2000) + 2000
    WHEN c.category_name = '大家电' THEN FLOOR(RAND() * 1000) + 800
  END,
  FLOOR(RAND() * 100) + 50,
  10,
  CASE 
    WHEN c.category_name = '智能手机' THEN 0.2
    WHEN c.category_name = '笔记本电脑' THEN 2.5
    WHEN c.category_name = '大家电' THEN 50
  END,
  CONCAT(b.brand_name, ' 新款产品，性能卓越'),
  CONCAT('<p>', b.brand_name, ' 官方正品，品质保证</p><p>详细参数：</p><ul><li>参数1</li><li>参数2</li></ul>'),
  CONCAT('/goods/main/', b.brand_name, '.jpg'),
  1,
  CASE WHEN RAND() > 0.7 THEN 1 ELSE 0 END,
  CASE WHEN RAND() > 0.7 THEN 1 ELSE 0 END,
  CASE WHEN RAND() > 0.7 THEN 1 ELSE 0 END,
  FLOOR(RAND() * 100),
  FLOOR(RAND() * 50),
  ROUND(RAND() * 2 + 3, 1)
FROM goods_categories c
CROSS JOIN brands b
WHERE c.category_level = 2
LIMIT 20;
```

### 5. 商品SKU数据填充
```sql
-- 为每个商品生成SKU
INSERT INTO goods_skus (goods_id, sku_sn, spec_data, price, original_price, cost_price, stock, warn_stock, weight, volume, image_url, status, sales)
SELECT 
  g.id,
  CONCAT(g.goods_sn, '-', FLOOR(RAND() * 100)),
  CASE 
    WHEN g.category_id = (SELECT id FROM goods_categories WHERE category_name = '智能手机') THEN 
      JSON_OBJECT(
        '1', (SELECT id FROM goods_spec_values WHERE spec_id = 1 ORDER BY RAND() LIMIT 1),
        '2', (SELECT id FROM goods_spec_values WHERE spec_id = 2 ORDER BY RAND() LIMIT 1),
        '3', (SELECT id FROM goods_spec_values WHERE spec_id = 3 ORDER BY RAND() LIMIT 1)
      )
    WHEN g.category_id = (SELECT id FROM goods_categories WHERE category_name = '笔记本电脑') THEN 
      JSON_OBJECT(
        '1', (SELECT id FROM goods_spec_values WHERE spec_id = 1 ORDER BY RAND() LIMIT 1),
        '2', (SELECT id FROM goods_spec_values WHERE spec_id = 2 ORDER BY RAND() LIMIT 1),
        '3', (SELECT id FROM goods_spec_values WHERE spec_id = 3 ORDER BY RAND() LIMIT 1)
      )
    ELSE 
      JSON_OBJECT(
        '1', (SELECT id FROM goods_spec_values WHERE spec_id = 1 ORDER BY RAND() LIMIT 1)
      )
  END,
  g.shop_price * (0.9 + RAND() * 0.2),
  g.market_price,
  g.cost_price,
  FLOOR(RAND() * 50) + 20,
  5,
  g.goods_weight * (0.8 + RAND() * 0.4),
  CASE 
    WHEN g.category_id = (SELECT id FROM goods_categories WHERE category_name = '智能手机') THEN 0.001
    WHEN g.category_id = (SELECT id FROM goods_categories WHERE category_name = '笔记本电脑') THEN 0.01
    ELSE 0.5
  END,
  CONCAT('/goods/sku/', g.id, '-', @sku_num:=@sku_num+1, '.jpg'),
  1,
  FLOOR(RAND() * 30)
FROM goods g
CROSS JOIN (SELECT @sku_num:=0) t
WHERE g.is_on_sale = 1;
```

## 三、订单数据填充

### 1. 订单数据填充
```sql
-- 为每个用户生成0-5个订单
INSERT INTO orders (order_sn, user_id, order_status, shipping_status, pay_status, consignee, country, province, city, district, address, mobile, postscript, shipping_fee, discount_fee, coupon_fee, integral_fee, order_amount, total_amount, tax_fee, service_fee, platform_discount, pay_time, shipping_time, confirm_time, transaction_id, shipping_company, shipping_no, source, is_deleted, created_at)
SELECT 
  CONCAT('ORDER', DATE_FORMAT(create_date, '%Y%m%d'), FLOOR(RAND() * 1000000)),
  u.id,
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
  a.receiver_name,
  '中国',
  a.province,
  a.city,
  a.district,
  a.detailed_address,
  a.receiver_phone,
  CASE WHEN RAND() > 0.8 THEN '请尽快发货' ELSE NULL END,
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
    WHEN create_date < DATE_SUB(NOW(), INTERVAL 1 DAY) THEN CONCAT('TRANS', FLOOR(RAND() * 1000000000))
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
    WHEN create_date < DATE_SUB(NOW(), INTERVAL 2 DAY) THEN CONCAT('SF', FLOOR(RAND() * 100000000))
    ELSE NULL
  END,
  FLOOR(RAND() * 4) + 1,
  0,
  create_date
FROM (
  SELECT 
    u.id,
    a.receiver_name,
    a.province,
    a.city,
    a.district,
    a.detailed_address,
    a.receiver_phone,
    DATE_ADD('2023-01-01', INTERVAL FLOOR(RAND() * 365) DAY) AS create_date,
    FLOOR(RAND() * 500) + 100 AS total_amount
  FROM users u
  LEFT JOIN user_addresses a ON u.id = a.user_id AND a.is_default = 1
  CROSS JOIN (SELECT 1 AS n UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) counts
  WHERE RAND() < 0.7 OR counts.n = 1
) u
ORDER BY create_date;
```

### 2. 订单商品数据填充
```sql
-- 为每个订单添加1-5个商品
INSERT INTO order_items (order_id, goods_id, goods_name, goods_sn, sku_id, sku_sn, spec_data, goods_number, market_price, goods_price, goods_attr, is_real, is_gift)
SELECT 
  o.id,
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
    WHEN g.category_id = (SELECT id FROM goods_categories WHERE category_name = '智能手机') THEN '颜色:黑色;内存:8GB;存储:128GB'
    WHEN g.category_id = (SELECT id FROM goods_categories WHERE category_name = '笔记本电脑') THEN '颜色:银色;内存:16GB;存储:512GB'
    ELSE '颜色:白色'
  END,
  1,
  CASE WHEN RAND() > 0.9 THEN 1 ELSE 0 END
FROM orders o
CROSS JOIN (
  SELECT g.*, gs.id AS sku_id, gs.spec_data, gs.price 
  FROM goods g
  JOIN goods_skus gs ON g.id = gs.goods_id
  WHERE g.is_on_sale = 1
  ORDER BY RAND()
  LIMIT 100
) g
WHERE RAND() < 0.3 OR NOT EXISTS (SELECT 1 FROM order_items WHERE order_id = o.id)
ORDER BY o.id, RAND()
LIMIT 500;
```

## 四、购物车数据填充

```sql
-- 为用户添加购物车商品
INSERT INTO cart (user_id, session_id, goods_id, sku_id, goods_name, quantity, price, market_price, spec_data, selected, created_at)
SELECT 
  u.id,
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
FROM users u
CROSS JOIN (
  SELECT g.*, gs.id AS sku_id, gs.spec_data, gs.price 
  FROM goods g
  JOIN goods_skus gs ON g.id = gs.goods_id
  WHERE g.is_on_sale = 1
  ORDER BY RAND()
  LIMIT 100
) g
WHERE RAND() < 0.4;
```

## 五、评价数据填充

```sql
-- 为已完成订单添加评价
INSERT INTO goods_comments (order_id, goods_id, user_id, username, content, comment_rank, is_anonymous, has_picture, reply_content, reply_time, created_at)
SELECT 
  oi.order_id,
  oi.goods_id,
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
FROM order_items oi
JOIN orders o ON oi.order_id = o.id
JOIN users u ON o.user_id = u.id
WHERE o.order_status = 3
AND RAND() < 0.7
ORDER BY RAND()
LIMIT 200;

-- 为部分评价添加图片
INSERT INTO comment_pictures (comment_id, pic_url, sort_order)
SELECT 
  c.id,
  CONCAT('/comments/', c.id, '-', p.seq, '.jpg'),
  p.seq
FROM goods_comments c
CROSS JOIN (
  SELECT 1 AS seq UNION SELECT 2 UNION SELECT 3
) p
WHERE c.has_picture = 1
AND p.seq <= FLOOR(RAND() * 3) + 1;
```

## 六、促销活动数据填充

### 1. 秒杀活动
```sql
INSERT INTO seckill_activity (name, start_time, end_time, status, limit_per_user, created_by, created_at, updated_at)
VALUES 
('618手机秒杀', '2023-06-18 00:00:00', '2023-06-18 23:59:59', 2, 1, 1, NOW(), NOW()),
('双十一数码秒杀', '2023-11-11 00:00:00', '2023-11-11 23:59:59', 1, 2, 1, NOW(), NOW());

-- 秒杀商品
INSERT INTO seckill_goods (activity_id, goods_id, sku_id, seckill_price, seckill_stock, original_price, limit_buy, sort_order)
SELECT 
  sa.id,
  g.id,
  gs.id,
  gs.price * 0.7,
  FLOOR(gs.stock * 0.3),
  gs.price,
  1,
  1
FROM seckill_activity sa
CROSS JOIN goods g
JOIN goods_skus gs ON g.id = gs.goods_id
WHERE g.is_on_sale = 1
AND RAND() < 0.3
ORDER BY RAND()
LIMIT 10;
```

### 2. 预售活动
```sql
INSERT INTO pre_sale_activity (name, start_time, end_time, delivery_date, deposit_amount, discount_amount, status, created_by, created_at, updated_at)
VALUES 
('新品预售', '2023-05-01 00:00:00', '2023-05-15 23:59:59', '2023-06-01', 100.00, 200.00, 2, 1, NOW(), NOW()),
('夏季家电预售', '2023-06-01 00:00:00', '2023-06-30 23:59:59', '2023-07-15', 200.00, 300.00, 1, 1, NOW(), NOW());

-- 预售商品
INSERT INTO pre_sale_goods (activity_id, goods_id, sku_id, pre_sale_price, original_price, pre_sale_stock, limit_buy, sort_order)
SELECT 
  pa.id,
  g.id,
  gs.id,
  gs.price * 0.8,
  gs.price,
  FLOOR(gs.stock * 0.5),
  1,
  1
FROM pre_sale_activity pa
CROSS JOIN goods g
JOIN goods_skus gs ON g.id = gs.goods_id
WHERE g.is_on_sale = 1
AND RAND() < 0.3
ORDER BY RAND()
LIMIT 10;
```

## 七、用户行为数据填充

```sql
-- 用户浏览记录
INSERT INTO user_behavior_log (user_id, behavior_type, page_url, goods_id, sku_id, stay_duration, device_info, ip_address, location, created_at)
SELECT 
  u.id,
  1,
  CONCAT('/goods/', g.id),
  g.id,
  gs.id,
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
FROM users u
CROSS JOIN goods g
JOIN goods_skus gs ON g.id = gs.goods_id
WHERE RAND() < 0.4
ORDER BY RAND()
LIMIT 500;

-- 用户加购行为
INSERT INTO user_behavior_log (user_id, behavior_type, page_url, goods_id, sku_id, device_info, ip_address, location, created_at)
SELECT 
  user_id,
  4,
  CONCAT('/goods/', goods_id),
  goods_id,
  sku_id,
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
  DATE_ADD(created_at, INTERVAL FLOOR(RAND() * 60) MINUTE)
FROM cart
WHERE RAND() < 0.8;
```

## 八、搜索关键词数据填充

```sql
INSERT INTO search_keyword_stats (keyword, search_count, click_count, result_count, last_search_time)
VALUES 
('手机', 1250, 980, 45, NOW()),
('笔记本电脑', 850, 620, 32, NOW()),
('智能手表', 620, 450, 28, NOW()),
('空调', 580, 420, 25, NOW()),
('冰箱', 520, 380, 22, NOW()),
('洗衣机', 480, 350, 20, NOW()),
('电视', 450, 320, 18, NOW()),
('耳机', 420, 300, 15, NOW()),
('路由器', 380, 280, 12, NOW()),
('打印机', 350, 250, 10, NOW());
```

## 九、库存预警数据填充

```sql
INSERT INTO stock_warning (sku_id, goods_id, current_stock, warning_stock, status, created_at)
SELECT 
  gs.id,
  g.id,
  gs.stock,
  gs.warn_stock,
  0,
  NOW()
FROM goods_skus gs
JOIN goods g ON gs.goods_id = g.id
WHERE gs.stock <= gs.warn_stock
AND RAND() < 0.5;
```

## 十、用户标签数据填充

```sql
-- 用户标签
INSERT INTO user_tags (tag_name, tag_type, tag_desc, created_at)
VALUES 
('高消费用户', 1, '月均消费超过5000元', NOW()),
('活跃用户', 1, '每周登录3次以上', NOW()),
('新用户', 1, '注册时间不足1个月', NOW()),
('潜在流失用户', 1, '30天未登录', NOW()),
('VIP用户', 1, '购买过VIP服务', NOW());

-- 用户标签关联
INSERT INTO user_tag_relation (user_id, tag_id, created_at)
SELECT 
  u.id,
  t.id,
  NOW()
FROM users u
CROSS JOIN user_tags t
WHERE 
  (t.tag_name = '高消费用户' AND u.id IN (SELECT user_id FROM orders GROUP BY user_id HAVING SUM(order_amount) > 5000)) OR
  (t.tag_name = '活跃用户' AND RAND() < 0.3) OR
  (t.tag_name = '新用户' AND u.register_time > DATE_SUB(NOW(), INTERVAL 1 MONTH)) OR
  (t.tag_name = '潜在流失用户' AND u.last_login_time < DATE_SUB(NOW(), INTERVAL 30 DAY)) OR
  (t.tag_name = 'VIP用户' AND RAND() < 0.1);
```

## 十一、物流轨迹数据填充

```sql
INSERT INTO logistics_tracking (logistics_no, status, status_desc, location, operator, contact, update_time, created_at)
SELECT 
  o.shipping_no,
  CASE FLOOR(RAND() * 5)
    WHEN 0 THEN '已揽收'
    WHEN 1 THEN '运输中'
    WHEN 2 THEN '到达分拣中心'
    WHEN 3 THEN '派送中'
    ELSE '已签收'
  END,
  CASE 
    WHEN status = '已揽收' THEN '快递员已揽收'
    WHEN status = '运输中' THEN '正在运输途中'
    WHEN status = '到达分拣中心' THEN '已到达目的地分拣中心'
    WHEN status = '派送中' THEN '快递员正在派送'
    ELSE '收件人已签收'
  END,
  CASE 
    WHEN status = '已揽收' THEN o.province
    WHEN status = '运输中' THEN 
      CASE FLOOR(RAND() * 3)
        WHEN 0 THEN '上海分拣中心'
        WHEN 1 THEN '武汉分拣中心'
        ELSE '广州分拣中心'
      END
    WHEN status = '到达分拣中心' THEN CONCAT(o.province, '分拣中心')
    WHEN status = '派送中' THEN CONCAT(o.city, o.district, '网点')
    ELSE o.address
  END,
  CASE 
    WHEN status = '已揽收' THEN '张师傅'
    WHEN status = '派送中' THEN '李师傅'
    WHEN status = '已签收' THEN '收件人'
    ELSE NULL
  END,
  CASE 
    WHEN status = '已揽收' THEN '13800001111'
    WHEN status = '派送中' THEN '13800002222'
    ELSE NULL
  END,
  CASE 
    WHEN status = '已揽收' THEN DATE_ADD(o.shipping_time, INTERVAL FLOOR(RAND() * 2) HOUR)
    WHEN status = '运输中' THEN DATE_ADD(o.shipping_time, INTERVAL FLOOR(RAND() * 12 + 2) HOUR)
    WHEN status = '到达分拣中心' THEN DATE_ADD(o.shipping_time, INTERVAL FLOOR(RAND() * 24 + 12) HOUR)
    WHEN status = '派送中' THEN DATE_ADD(o.shipping_time, INTERVAL FLOOR(RAND() * 12 + 36) HOUR)
    ELSE o.confirm_time
  END,
  NOW()
FROM orders o
WHERE o.shipping_no IS NOT NULL
AND o.shipping_time IS NOT NULL
ORDER BY RAND()
LIMIT 100;
```

## 十二、促销活动数据填充

```sql
-- 促销活动
INSERT INTO promotion_activity (name, type, start_time, end_time, status, description, scope_type, created_by, created_at, updated_at)
VALUES 
('618大促', 1, '2023-06-01 00:00:00', '2023-06-20 23:59:59', 2, '618年中大促活动', 1, 1, NOW(), NOW()),
('双十一预售', 2, '2023-10-20 00:00:00', '2023-11-11 23:59:59', 1, '双十一预售活动', 2, 1, NOW(), NOW()),
('年终清仓', 3, '2023-12-01 00:00:00', '2023-12-31 23:59:59', 1, '年终清仓特卖', 3, 1, NOW(), NOW());

-- 促销规则
INSERT INTO promotion_rule (activity_id, condition_type, condition_value, discount_type, discount_value, gift_goods_id, sort_order)
VALUES 
(1, 1, 300.00, 1, 30.00, NULL, 1),
(1, 1, 500.00, 1, 80.00, NULL, 2),
(1, 1, 1000.00, 1, 150.00, NULL, 3),
(2, 1, 200.00, 2, 0.90, NULL, 1),
(3, 2, 2.00, 3, 0.00, (SELECT id FROM goods ORDER BY RAND() LIMIT 1), 1);

-- 促销范围
INSERT INTO promotion_scope (activity_id, target_id, target_type)
SELECT 
  pa.id,
  g.id,
  1
FROM promotion_activity pa
CROSS JOIN goods g
WHERE pa.scope_type = 2
AND RAND() < 0.3
ORDER BY RAND()
LIMIT 20;

INSERT INTO promotion_scope (activity_id, target_id, target_type)
SELECT 
  pa.id,
  gc.id,
  2
FROM promotion_activity pa
CROSS JOIN goods_categories gc
WHERE pa.scope_type = 3
AND gc.category_level = 2
AND RAND() < 0.5
ORDER BY RAND()
LIMIT 10;
```

## 使用说明

1. 执行顺序建议按照本脚本的章节顺序执行
2. 数据量可根据需要调整LIMIT值
3. 时间范围可根据实际需求调整
4. 关联数据会自动关联已存在的数据
5. 随机数据会保持业务逻辑合理性

此脚本为电商系统提供了完整的测试数据，覆盖了用户、商品、订单、促销等核心业务模块，可以用于系统测试、演示和性能评估。