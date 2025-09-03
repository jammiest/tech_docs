# PHP 中的正则表达式实现

> PHP 提供了两套正则表达式函数库：PCRE (Perl Compatible Regular Expressions) 和 POSIX 扩展。本节将重点介绍 PCRE 函数库，这是 PHP 中最常用且功能最强大的正则表达式实现。

## 基本语法

### 1. 分隔符

PHP 中的正则表达式需要包含在分隔符中，常用分隔符包括：

```php
/pattern/    # 最常见
#pattern#    # 也常用
~pattern~    # 当模式包含/时使用
```

### 2. 模式修饰符

| 修饰符 | 描述 | 示例 |
|--------|------|------|
| `i` | 忽略大小写 | `/abc/i` 匹配 "abc"、"ABC" |
| `m` | 多行模式 | `/^abc/m` 匹配每行开头的 "abc" |
| `s` | 使 `.` 匹配包括换行符在内的所有字符 | `/a.b/s` 匹配 "a\nb" |
| `x` | 忽略模式中的空白和注释 | `/a b c/x` 匹配 "abc" |
| `u` | Unicode 模式 | `/^\p{L}+$/u` 匹配Unicode字母 |

## 核心函数

### 1. 匹配函数

| 函数 | 描述 | 返回值 |
|------|------|--------|
| `preg_match()` | 执行匹配 | 匹配次数 (0或1) |
| `preg_match_all()` | 执行全局匹配 | 匹配次数 |
| `preg_grep()` | 返回匹配模式的数组元素 | 匹配的数组 |

### 2. 替换函数

| 函数 | 描述 | 返回值 |
|------|------|--------|
| `preg_replace()` | 执行正则替换 | 替换后的字符串 |
| `preg_filter()` | 类似于 `preg_replace()`，但只返回匹配项 | 过滤后的数组 |
| `preg_replace_callback()` | 使用回调函数进行替换 | 替换后的字符串 |

### 3. 分割函数

| 函数 | 描述 | 返回值 |
|------|------|--------|
| `preg_split()` | 用正则表达式分割字符串 | 分割后的数组 |

## 基本使用示例

### 1. 简单匹配

```php
$pattern = '/\d+/';
$subject = 'There are 123 apples';

if (preg_match($pattern, $subject, $matches)) {
    echo "Found: " . $matches[0];  // Found: 123
}
```

### 2. 全局匹配

```php
$pattern = '/\d+/';
$subject = '1 apple, 2 oranges, 3 bananas';

preg_match_all($pattern, $subject, $matches);
print_r($matches[0]);  // Array ( [0] => 1 [1] => 2 [2] => 3 )
```

### 3. 替换操作

```php
$pattern = '/\d+/';
$replacement = '#';
$subject = '1 apple, 2 oranges, 3 bananas';

$result = preg_replace($pattern, $replacement, $subject);
echo $result;  // # apple, # oranges, # bananas
```

### 4. 分割字符串

```php
$pattern = '/,\s*/';
$subject = 'apple, orange, banana';

$parts = preg_split($pattern, $subject);
print_r($parts);  // Array ( [0] => apple [1] => orange [2] => banana )
```

## 高级特性

### 1. 命名捕获组

```php
$pattern = '/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/';
$subject = '2023-05-15';

if (preg_match($pattern, $subject, $matches)) {
    echo $matches['year'];   // 2023
    echo $matches['month'];  // 05
    echo $matches['day'];    // 15
}
```

### 2. 非捕获组

```php
$pattern = '/(?:Mr|Ms|Mrs)\. (\w+)/';
$subject = 'Mr. Smith';

if (preg_match($pattern, $subject, $matches)) {
    echo $matches[1];  // Smith (只捕获名字)
}
```

### 3. 断言

```php
// 正向先行断言
$pattern = '/\w+(?=;)/';
$subject = 'apple; orange; banana';

preg_match_all($pattern, $subject, $matches);
print_r($matches[0]);  // Array ( [0] => apple [1] => orange [2] => banana )

// 正向后行断言
$pattern = '/(?<=\$)\d+/';
$subject = 'Price: $100';

preg_match($pattern, $subject, $matches);
echo $matches[0];  // 100
```

### 4. 递归模式

```php
$pattern = '/\(((?:[^()]|(?R))*)\)/';
$subject = 'a (b (c) d) e';

preg_match_all($pattern, $subject, $matches);
print_r($matches[1]);  // Array ( [0] => b (c) d [1] => c )
```

## 实际应用示例

### 1. 表单验证

```php
// 验证邮箱
function isValidEmail($email) {
    return preg_match('/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/', $email);
}

// 验证手机号（中国）
function isValidChinesePhone($phone) {
    return preg_match('/^1[3-9]\d{9}$/', $phone);
}

// 验证密码强度（至少8字符，包含大小写字母和数字）
function isStrongPassword($password) {
    return preg_match('/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/', $password);
}
```

### 2. 数据提取

```php
// 从文本中提取所有URL
function extractUrls($text) {
    preg_match_all('/https?:\/\/[^\s\/$.?#].[^\s]*/', $text, $matches);
    return $matches[0];
}

// 从HTML中提取所有图片src
function extractImageSrcs($html) {
    preg_match_all('/<img[^>]+src="([^"]+)"/', $html, $matches);
    return $matches[1];
}

// 提取CSS中的颜色值
function extractColors($css) {
    preg_match_all('/#(?:[a-fA-F0-9]{6}|[a-fA-F0-9]{3})\b|rgb\(\s*\d{1,3}\s*,\s*\d{1,3}\s*,\s*\d{1,3}\s*\)/', $css, $matches);
    return $matches[0];
}
```

### 3. 文本处理

```php
// 移除HTML标签
function removeHtmlTags($html) {
    return preg_replace('/<[^>]+>/', '', $html);
}

// 格式化数字（添加千位分隔符）
function formatNumber($number) {
    return preg_replace('/\B(?=(\d{3})+(?!\d))/', ',', $number);
}

// 驼峰命名转下划线命名
function camelToSnake($name) {
    return strtolower(preg_replace('/([a-z])([A-Z])/', '$1_$2', $name));
}
```

## 回调替换

### 1. 基本用法

```php
$text = "April 15, 2003";
$pattern = '/(\w+) (\d+), (\d+)/i';
$replacement = '${1}1,$3';

echo preg_replace($pattern, $replacement, $text);  // April1,2003
```

### 2. 使用回调函数

```php
$text = "Hello world";
$pattern = '/\b\w+\b/';

$result = preg_replace_callback($pattern, function($matches) {
    return strtoupper($matches[0]);
}, $text);

echo $result;  // HELLO WORLD
```

## 错误处理

### 1. 获取错误信息

```php
preg_match('/invalid pattern/', 'test');

if (preg_last_error() !== PREG_NO_ERROR) {
    echo "Regex error: " . preg_last_error_msg();
    // 可能输出: Regex error: Compilation failed: missing terminating ] for character class
}
```

### 2. 常见错误代码

| 常量 | 描述 |
|------|------|
| `PREG_NO_ERROR` | 没有错误 |
| `PREG_INTERNAL_ERROR` | 内部PCRE错误 |
| `PREG_BACKTRACK_LIMIT_ERROR` | 回溯限制超出 |
| `PREG_RECURSION_LIMIT_ERROR` | 递归限制超出 |
| `PREG_BAD_UTF8_ERROR` | 无效的UTF-8数据 |
| `PREG_BAD_UTF8_OFFSET_ERROR` | UTF-8偏移量无效 |

## 性能优化

### 1. 预编译模式

```php
// 不好的做法（每次调用都编译正则）
function badCheck($input) {
    return preg_match('/\d+/', $input);
}

// 好的做法（使用常量存储编译后的模式）
define('NUMBER_PATTERN', '/\d+/');
function goodCheck($input) {
    return preg_match(NUMBER_PATTERN, $input);
}
```

### 2. 避免回溯爆炸

```php
// 危险的正则表达式（可能导致回溯爆炸）
$dangerous = '/(a+)+b/';

// 更安全的替代方案
$safe = '/a+b/';
```

### 3. 使用具体字符类

```php
// 不好
$bad = '/.*abc/';

// 更好
$good = '/[^abc]*abc/';
```

## 常见问题与解决方案

### 1. 转义字符处理

```php
// 匹配点号
$dotPattern = '/\./';

// 匹配反斜杠
$backslashPattern = '/\\\\/';
```

### 2. Unicode字符处理

```php
// 匹配中文字符
$chinesePattern = '/[\x{4e00}-\x{9fa5}]+/u';

// 匹配emoji
$emojiPattern = '/[\x{1F600}-\x{1F64F}]/u';
```

### 3. 多行匹配问题

```php
$text = "first line\nsecond line\nthird line";

// 不使用多行模式
preg_match_all('/^.*$/', $text, $matches);  // 不匹配

// 使用多行模式
preg_match_all('/^.*$/m', $text, $matches);  // 匹配所有行
```

## PHP 7+ 新特性

### 1. preg_replace_callback_array()

PHP 7+ 引入了 `preg_replace_callback_array()`，允许对不同的模式使用不同的回调函数：

```php
$subject = 'Hello world';

$result = preg_replace_callback_array(
    [
        '/\bHello\b/' => function ($match) {
            return 'Hi';
        },
        '/\bworld\b/' => function ($match) {
            return 'there';
        }
    ],
    $subject
);

echo $result;  // Hi there
```

### 2. 命名子组引用

PHP 7+ 增强了命名子组的引用方式：

```php
$pattern = '/(?<name>\w+) (?<age>\d+)/';
$replacement = 'Name: ${name}, Age: ${age}';
$subject = 'John 30';

echo preg_replace($pattern, $replacement, $subject);  // Name: John, Age: 30
```

## 总结

PHP 中的 PCRE 正则表达式实现提供了强大的文本处理能力：

- **核心函数**：`preg_match()`, `preg_replace()`, `preg_split()` 等
- **高级特性**：命名捕获组、递归模式、断言等
- **实际应用**：表单验证、数据提取、文本处理
- **性能优化**：预编译模式、避免回溯爆炸
- **错误处理**：`preg_last_error()` 和 `preg_last_error_msg()`

> 提示：对于复杂的文本处理需求，可以考虑结合多个简单的正则表达式分步骤处理。PHP 的 PCRE 实现非常强大，但在处理复杂嵌套结构时可能不如专门的解析器高效。