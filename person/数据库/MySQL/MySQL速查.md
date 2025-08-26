# MySQL 语句速查表

## 数据库操作
```sql
CREATE DATABASE dbname;                    -- 创建数据库
DROP DATABASE dbname;                      -- 删除数据库
USE dbname;                                -- 选择数据库
SHOW DATABASES;                            -- 显示所有数据库
SHOW TABLES;                               -- 显示当前数据库的所有表
```

## 表操作
```sql
CREATE TABLE table_name (                  -- 创建表
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DROP TABLE table_name;                     -- 删除表
ALTER TABLE table_name ADD column_name INT; -- 添加列
ALTER TABLE table_name DROP COLUMN column_name; -- 删除列
ALTER TABLE table_name MODIFY column_name VARCHAR(100); -- 修改列类型
ALTER TABLE table_name RENAME TO new_name; -- 重命名表
DESC table_name;                           -- 查看表结构
```

## 数据操作 (CRUD)
```sql
-- 插入数据
INSERT INTO table_name (col1, col2) VALUES (val1, val2);
INSERT INTO table_name VALUES (val1, val2, val3);

-- 查询数据
SELECT * FROM table_name;
SELECT col1, col2 FROM table_name;
SELECT DISTINCT col1 FROM table_name;      -- 去重查询

-- 更新数据
UPDATE table_name SET col1 = val1 WHERE condition;

-- 删除数据
DELETE FROM table_name WHERE condition;
TRUNCATE TABLE table_name;                 -- 清空表（更快）
```

## 条件查询
```sql
SELECT * FROM table_name WHERE age > 18;
SELECT * FROM table_name WHERE name LIKE 'J%';     -- J开头
SELECT * FROM table_name WHERE name LIKE '%n';      -- n结尾
SELECT * FROM table_name WHERE name LIKE '%oh%';    -- 包含oh
SELECT * FROM table_name WHERE age BETWEEN 18 AND 30; -- 范围查询
SELECT * FROM table_name WHERE age IN (18, 20, 25); -- 在列表中
SELECT * FROM table_name WHERE email IS NULL;       -- 为空
SELECT * FROM table_name WHERE email IS NOT NULL;   -- 不为空
```

## 排序和限制
```sql
SELECT * FROM table_name ORDER BY age ASC;      -- 升序
SELECT * FROM table_name ORDER BY age DESC;     -- 降序
SELECT * FROM table_name ORDER BY age DESC, name ASC; -- 多列排序
SELECT * FROM table_name LIMIT 10;              -- 前10条
SELECT * FROM table_name LIMIT 5, 10;           -- 从第6条开始取10条
```

## 聚合函数
```sql
SELECT COUNT(*) FROM table_name;                -- 总行数
SELECT COUNT(DISTINCT age) FROM table_name;     -- 去重计数
SELECT SUM(age) FROM table_name;                -- 求和
SELECT AVG(age) FROM table_name;                -- 平均值
SELECT MAX(age) FROM table_name;                -- 最大值
SELECT MIN(age) FROM table_name;                -- 最小值
```

## 分组查询
```sql
SELECT age, COUNT(*) FROM table_name GROUP BY age;
SELECT age, COUNT(*) FROM table_name 
GROUP BY age HAVING COUNT(*) > 5;               -- 分组后过滤
```

## 连接查询
```sql
-- 内连接
SELECT * FROM table1 
INNER JOIN table2 ON table1.id = table2.table1_id;

-- 左连接
SELECT * FROM table1 
LEFT JOIN table2 ON table1.id = table2.table1_id;

-- 右连接
SELECT * FROM table1 
RIGHT JOIN table2 ON table1.id = table2.table1_id;

-- 全外连接
SELECT * FROM table1 
LEFT JOIN table2 ON table1.id = table2.table1_id
UNION
SELECT * FROM table1 
RIGHT JOIN table2 ON table1.id = table2.table1_id;
```

## 子查询
```sql
SELECT * FROM table_name 
WHERE age > (SELECT AVG(age) FROM table_name);

SELECT * FROM table1 
WHERE id IN (SELECT table1_id FROM table2 WHERE condition);

SELECT * FROM (SELECT * FROM table_name WHERE age > 18) AS temp;
```

## 索引操作
```sql
CREATE INDEX idx_name ON table_name (column_name); -- 创建索引
CREATE UNIQUE INDEX idx_name ON table_name (column_name); -- 唯一索引
DROP INDEX idx_name ON table_name;               -- 删除索引
SHOW INDEX FROM table_name;                      -- 查看索引
```

## 事务处理
```sql
START TRANSACTION;                              -- 开始事务
INSERT INTO table_name VALUES (1, 'John');
UPDATE table_name SET age = 25 WHERE id = 1;
COMMIT;                                         -- 提交事务
ROLLBACK;                                       -- 回滚事务

-- 设置保存点
SAVEPOINT savepoint_name;
ROLLBACK TO savepoint_name;
```

## 用户和权限
```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password'; -- 创建用户
DROP USER 'username'@'host';                         -- 删除用户
GRANT ALL PRIVILEGES ON dbname.* TO 'username'@'host'; -- 授权
REVOKE ALL PRIVILEGES ON dbname.* FROM 'username'@'host'; -- 撤销权限
FLUSH PRIVILEGES;                                    -- 刷新权限
SHOW GRANTS FOR 'username'@'host';                   -- 查看权限
```

## 常用函数
```sql
-- 字符串函数
SELECT CONCAT('Hello', ' ', 'World');              -- 字符串连接
SELECT SUBSTRING('Hello', 2, 3);                   -- 子字符串
SELECT LENGTH('Hello');                            -- 字符串长度
SELECT UPPER('hello');                             -- 转大写
SELECT LOWER('HELLO');                             -- 转小写
SELECT TRIM('  hello  ');                          -- 去除空格

-- 数值函数
SELECT ROUND(3.14159, 2);                          -- 四舍五入
SELECT CEIL(3.14);                                 -- 向上取整
SELECT FLOOR(3.14);                                -- 向下取整
SELECT ABS(-10);                                   -- 绝对值
SELECT RAND();                                     -- 随机数

-- 日期函数
SELECT NOW();                                      -- 当前日期时间
SELECT CURDATE();                                  -- 当前日期
SELECT CURTIME();                                  -- 当前时间
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');    -- 日期格式化
SELECT DATEDIFF('2023-12-31', '2023-01-01');       -- 日期差
```

## 数据导入导出
```sql
-- 导出数据
SELECT * FROM table_name INTO OUTFILE '/tmp/data.csv'
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n';

-- 导入数据
LOAD DATA INFILE '/tmp/data.csv' INTO TABLE table_name
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

## 实用技巧
```sql
-- 复制表结构
CREATE TABLE new_table LIKE original_table;

-- 复制表结构和数据
CREATE TABLE new_table AS SELECT * FROM original_table;

-- 批量插入
INSERT INTO table_name (col1, col2) VALUES 
(val1, val2),
(val3, val4),
(val5, val6);

-- 存在则更新，不存在则插入
INSERT INTO table_name (id, name) VALUES (1, 'John')
ON DUPLICATE KEY UPDATE name = 'John';

-- 使用CASE语句
SELECT name, 
       CASE 
           WHEN age < 18 THEN '未成年'
           WHEN age < 60 THEN '成年人'
           ELSE '老年人'
       END AS age_group
FROM table_name;
```