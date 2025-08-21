# # MySQL 数据类型及操作指南

## 一、MySQL 数据类型详解

### 1. 数值类型

#### 整数类型
| 类型     | 字节 | 有符号范围 | 无符号范围 | 示例                    |
| -------- | ---- | ---------- | ---------- | ----------------------- |
| TINYINT  | 1    | -128~127   | 0~255      | `age TINYINT UNSIGNED`  |
| SMALLINT | 2    | ±32K       | 0~65K      | `count SMALLINT`        |
| INT      | 4    | ±2B        | 0~4B       | `id INT AUTO_INCREMENT` |
| BIGINT   | 8    | ±9E18      | 0~18E18    | `user_id BIGINT`        |

#### 小数类型
```sql
FLOAT(7,4)      -- 单精度浮点，4字节
DOUBLE(15,8)    -- 双精度浮点，8字节
DECIMAL(10,2)   -- 精确小数，适合金额
```

### 2. 字符串类型

#### 文本类型
```sql
CHAR(10)        -- 定长，适合短字符串（如性别）
VARCHAR(255)    -- 变长，最大65535字节
TEXT            -- 长文本，最大64KB
LONGTEXT        -- 超大文本，最大4GB
```

#### 二进制类型
```sql
BINARY(16)      -- 定长二进制（如MD5）
VARBINARY(255)  -- 变长二进制
BLOB            -- 二进制大对象
```

### 3. 日期时间类型

```sql
DATE            -- 'YYYY-MM-DD'
TIME            -- 'HH:MM:SS'
DATETIME        -- 'YYYY-MM-DD HH:MM:SS'（1000-9999年）
TIMESTAMP       -- 时间戳（1970-2038年）
YEAR(4)         -- 年份
```

### 4. 特殊类型

```sql
ENUM('男','女')             -- 单选枚举
SET('阅读','音乐','运动')    -- 多选集合
JSON                       -- JSON数据
GEOMETRY                   -- 空间数据
```

## 二、数据类型操作实战

### 1. 数值类型操作

#### 基本运算
```sql
UPDATE products 
SET price = price * 0.9  -- 打9折
WHERE category = '电子';

SELECT AVG(age) FROM users;  -- 平均年龄
```

#### 类型转换
```sql
SELECT CAST('123' AS SIGNED);  -- 字符串转整数
SELECT CONVERT('2023-01-01', DATE);  -- 字符串转日期
```

### 2. 字符串操作

#### 常用函数
```sql
-- 连接字符串
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;

-- 子字符串
SELECT SUBSTRING(description, 1, 100) AS brief FROM articles;

-- 大小写转换
SELECT UPPER(username), LOWER(email) FROM users;

-- 去除空格
SELECT TRIM(comment) FROM reviews;
```

#### 模式匹配
```sql
-- LIKE模糊查询
SELECT * FROM products 
WHERE name LIKE '%手机%';

-- 正则表达式
SELECT * FROM logs 
WHERE message REGEXP 'error|fail';
```

### 3. 日期时间操作

#### 日期计算
```sql
-- 增加天数
SELECT DATE_ADD(NOW(), INTERVAL 7 DAY);

-- 计算差值
SELECT DATEDIFF('2023-12-31', '2023-01-01');

-- 提取日期部分
SELECT YEAR(created_at), MONTH(created_at) FROM orders;
```

#### 日期格式化
```sql
SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日 %H:%i:%s');
```

### 4. JSON 类型操作

#### 创建与查询
```sql
-- 创建JSON字段
ALTER TABLE products ADD COLUMN attributes JSON;

-- 插入JSON数据
UPDATE products SET attributes = '{"color":"黑色","weight":"1.5kg"}' 
WHERE id = 1;

-- 查询JSON属性
SELECT name, attributes->'$.color' AS color FROM products;
```

#### JSON函数
```sql
-- 修改JSON
UPDATE products 
SET attributes = JSON_SET(attributes, '$.memory', '8GB') 
WHERE id = 2;

-- 检查键是否存在
SELECT name FROM products 
WHERE JSON_CONTAINS_PATH(attributes, 'one', '$.color');
```

## 三、数据类型最佳实践

### 1. 选择原则

1. **最小够用原则**：
   - 年龄用`TINYINT UNSIGNED`而非`INT`
   - 短字符串用`CHAR`而非`VARCHAR`

2. **精确性原则**：
   - 金融数据必须用`DECIMAL`
   - 时间记录用`DATETIME`或`TIMESTAMP`

3. **扩展性原则**：
   - 可能扩展的枚举用`VARCHAR`而非`ENUM`
   - 不确定长度的文本用`TEXT`

### 2. 性能优化

1. **索引优化**：
   ```sql
   -- 为常用查询字段添加索引
   ALTER TABLE users ADD INDEX idx_email (email);
   
   -- 前缀索引
   ALTER TABLE articles ADD INDEX idx_title (title(20));
   ```

2. **存储优化**：
   ```sql
   -- 使用压缩文本
   INSERT INTO logs (content) VALUES (COMPRESS('长文本内容'));
   
   -- 垂直分表（大字段分离）
   CREATE TABLE product_details (
     product_id INT PRIMARY KEY,
     long_description TEXT,
     FOREIGN KEY (product_id) REFERENCES products(id)
   );
   ```

## 四、高级数据类型应用

### 1. 空间数据示例

```sql
-- 创建空间表
CREATE TABLE locations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    position POINT SRID 4326,
    SPATIAL INDEX(position)
);

-- 插入空间数据
INSERT INTO locations (name, position) 
VALUES ('天安门', ST_PointFromText('POINT(116.404 39.915)'));

-- 空间查询
SELECT name FROM locations 
WHERE ST_Distance_Sphere(position, ST_Point(116.4, 39.9)) < 1000;
```

### 2. 生成列示例

```sql
-- 虚拟生成列
ALTER TABLE orders 
ADD COLUMN total_amount DECIMAL(12,2) 
GENERATED ALWAYS AS (quantity * unit_price) VIRTUAL;

-- 存储生成列
ALTER TABLE products 
ADD COLUMN profit DECIMAL(10,2) 
GENERATED ALWAYS AS (price - cost) STORED;
```

## 五、常见问题解决方案

### 1. 日期处理问题

```sql
-- 时区转换
SELECT CONVERT_TZ(created_at, '+00:00', '+08:00') FROM events;

-- 计算工作日（排除周末）
SELECT COUNT(*) FROM calendar 
WHERE date BETWEEN '2023-01-01' AND '2023-01-31' 
AND DAYOFWEEK(date) NOT IN (1,7);
```

### 2. JSON性能优化

```sql
-- 创建JSON索引（MySQL 8.0+）
ALTER TABLE products 
ADD INDEX idx_color ((CAST(attributes->'$.color' AS CHAR(20))));

-- 物化JSON路径
ALTER TABLE products 
ADD COLUMN color VARCHAR(20) 
GENERATED ALWAYS AS (attributes->'$.color') STORED,
ADD INDEX idx_color (color);
```

### 3. 大文本处理

```sql
-- 分页查询大文本
SELECT id, SUBSTRING(content, 1, 200) AS preview 
FROM articles 
ORDER BY id LIMIT 10 OFFSET 20;

-- 全文检索（需全文索引）
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('数据库' IN NATURAL LANGUAGE MODE);
```

通过合理选择数据类型和掌握相关操作技巧，可以显著提升MySQL数据库的性能和可靠性。建议根据实际业务需求选择最适合的数据类型，并定期优化表结构。