# PHP PDO_MySQL 扩展详解

PDO (PHP Data Objects) 是 PHP 提供的数据库访问抽象层，而 PDO_MySQL 是 PDO 的 MySQL 驱动实现。相比 MySQLi，PDO 提供了更统一的接口来访问不同类型的数据库。

## 基本使用

### 1. 连接数据库

```php
<?php
try {
    // 创建PDO连接
    $pdo = new PDO(
        'mysql:host=localhost;dbname=database;charset=utf8mb4',
        'username',
        'password',
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
        ]
    );
} catch (PDOException $e) {
    die("连接失败: " . $e->getMessage());
}
```

### 2. 执行简单查询

```php
// 执行查询
$stmt = $pdo->query("SELECT * FROM users");

// 获取所有结果
$users = $stmt->fetchAll();

foreach ($users as $user) {
    echo $user['username'] . "<br>";
}
```

## 预处理语句（防止SQL注入）

```php
// 准备预处理语句
$stmt = $pdo->prepare("INSERT INTO users (username, email) VALUES (:username, :email)");

// 绑定参数并执行
$stmt->execute([
    ':username' => 'john_doe',
    ':email' => 'john@example.com'
]);

echo "插入的行数: " . $stmt->rowCount();
```

## 事务处理

```php
try {
    // 开始事务
    $pdo->beginTransaction();
    
    // 执行多个操作
    $pdo->exec("UPDATE accounts SET balance = balance - 100 WHERE user_id = 1");
    $pdo->exec("UPDATE accounts SET balance = balance + 100 WHERE user_id = 2");
    
    // 提交事务
    $pdo->commit();
    echo "事务成功完成";
} catch (Exception $e) {
    // 回滚事务
    $pdo->rollBack();
    echo "事务失败: " . $e->getMessage();
}
```

## 常用方法

| 方法 | 描述 |
|------|------|
| `query()` | 执行SQL查询并返回结果集 |
| `exec()` | 执行SQL语句并返回受影响的行数 |
| `prepare()` | 准备预处理语句 |
| `execute()` | 执行预处理语句 |
| `fetch()` | 获取下一行 |
| `fetchAll()` | 获取所有结果行 |
| `fetchColumn()` | 获取结果中的下一行的单个列 |
| `rowCount()` | 返回受影响的行数 |
| `lastInsertId()` | 返回最后插入行的ID |

## 获取数据的多种方式

```php
$stmt = $pdo->query("SELECT * FROM users");

// 获取关联数组
$row = $stmt->fetch(PDO::FETCH_ASSOC);

// 获取数字索引数组
$row = $stmt->fetch(PDO::FETCH_NUM);

// 获取对象
$row = $stmt->fetch(PDO::FETCH_OBJ);

// 获取所有结果作为关联数组
$allRows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

## 错误处理

```php
// 设置错误模式为异常
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

try {
    $stmt = $pdo->prepare("SELECT * FROM non_existent_table");
    $stmt->execute();
} catch (PDOException $e) {
    echo "数据库错误: " . $e->getMessage();
    // 记录错误日志
    error_log($e->getMessage());
}
```

## 最佳实践

1. **总是使用预处理语句**来防止SQL注入
2. **设置字符集**（推荐utf8mb4）
3. **使用异常处理**错误
4. **关闭持久连接**在不需要时
5. **限制SELECT查询**，避免获取不必要的数据
6. **使用命名参数**提高代码可读性

## PDO 与 MySQLi 比较

| 特性 | PDO | MySQLi |
|------|-----|--------|
| 数据库支持 | 多种数据库 | 仅MySQL |
| 预处理语句 | 支持 | 支持 |
| 命名参数 | 支持 | 不支持 |
| 事务 | 支持 | 支持 |
| 存储过程 | 支持 | 支持更好 |
| 面向对象 | 完全 | 完全 |
| 过程化接口 | 有限 | 完全 |

PDO_MySQL 提供了更灵活、更安全的数据库访问方式，特别是当项目可能需要支持多种数据库时，PDO 是更好的选择。