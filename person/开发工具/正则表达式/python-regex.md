# Python 中的正则表达式实现

> Python 通过 `re` 模块提供了完整的正则表达式支持，包括模式匹配、搜索替换等操作。本节将详细介绍 Python 中正则表达式的使用方法和特性。

## 基本用法

### 导入 re 模块

```python
import re
```

### 常用函数

| 函数 | 描述 | 示例 |
|------|------|------|
| `re.match()` | 从字符串开头匹配模式 | `re.match(r'\d+', '123abc')` |
| `re.search()` | 扫描整个字符串寻找匹配 | `re.search(r'\d+', 'abc123')` |
| `re.findall()` | 返回所有匹配的子串列表 | `re.findall(r'\d+', 'a1b2c3')` |
| `re.finditer()` | 返回匹配结果的迭代器 | `re.finditer(r'\d+', 'a1b2c3')` |
| `re.sub()` | 替换匹配的子串 | `re.sub(r'\d+', '#', 'a1b2')` |
| `re.split()` | 按模式分割字符串 | `re.split(r'\d+', 'a1b2c3')` |

## 正则表达式对象

### 编译正则表达式

```python
pattern = re.compile(r'\d+')  # 编译正则表达式对象
pattern.match('123abc')       # 使用编译后的对象
```

### 正则对象方法

| 方法 | 描述 | 示例 |
|------|------|------|
| `match()` | 从开头匹配 | `pattern.match('123abc')` |
| `search()` | 搜索匹配 | `pattern.search('abc123')` |
| `findall()` | 查找所有匹配 | `pattern.findall('a1b2c3')` |
| `finditer()` | 返回匹配迭代器 | `pattern.finditer('a1b2c3')` |
| `sub()` | 替换匹配 | `pattern.sub('#', 'a1b2')` |
| `split()` | 分割字符串 | `pattern.split('a1b2c3')` |

## 匹配结果对象

匹配操作返回的 Match 对象包含以下常用方法和属性：

```python
match = re.search(r'(\d+).(\d+)', '3.14 is pi')
```

### Match 对象方法

| 方法/属性 | 描述 | 示例 |
|-----------|------|------|
| `group()` | 返回匹配的字符串 | `match.group()` → '3.14' |
| `group(n)` | 返回第n个分组 | `match.group(1)` → '3' |
| `groups()` | 返回所有分组元组 | `match.groups()` → ('3', '14') |
| `start()` | 匹配开始位置 | `match.start()` → 0 |
| `end()` | 匹配结束位置 | `match.end()` → 4 |
| `span()` | 匹配范围 (start, end) | `match.span()` → (0, 4) |

## 高级特性

### 1. 命名捕获组

```python
match = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})', '2023-05')
print(match.group('year'))   # '2023'
print(match.group('month'))  # '05'
```

### 2. 非捕获组

```python
# 普通捕获组
re.findall(r'(a|b)c', 'ac bc')  # ['a', 'b']

# 非捕获组
re.findall(r'(?:a|b)c', 'ac bc')  # ['ac', 'bc']
```

### 3. 注释模式 (verbose)

```python
pattern = re.compile(r"""
    ^                  # 字符串开始
    (\d{3})           # 区号
    -                 # 分隔符
    (\d{3})           # 前缀
    -                 # 分隔符
    (\d{4})           # 线路号
    $                  # 字符串结束
""", re.VERBOSE)
```

### 4. 多行模式

```python
text = """first line
second line
third line"""

# 不使用多行模式
re.findall('^.*$', text)  # ['first line\nsecond line\nthird line']

# 使用多行模式
re.findall('^.*$', text, re.MULTILINE)  # ['first line', 'second line', 'third line']
```

### 5. 替换回调函数

```python
def to_upper(match):
    return match.group().upper()

re.sub(r'[a-z]+', to_upper, 'hello world')  # 'HELLO WORLD'
```

## 实际应用示例

### 1. 数据验证

```python
# 验证邮箱地址
def is_valid_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

# 验证手机号码（中国）
def is_valid_phone(phone):
    pattern = r'^1[3-9]\d{9}$'
    return bool(re.match(pattern, phone))
```

### 2. 数据提取

```python
# 从文本中提取所有URL
def extract_urls(text):
    pattern = r'https?://[^\s/$.?#].[^\s]*'
    return re.findall(pattern, text)

# 从HTML中提取所有链接
def extract_links(html):
    pattern = r'<a\s+href=[^"\']+["\']'
    return re.findall(pattern, html)
```

### 3. 文本处理

```python
# 移除HTML标签
def remove_html_tags(html):
    pattern = r'<[^>]+>'
    return re.sub(pattern, '', html)

# 格式化数字（添加千位分隔符）
def format_number(num):
    pattern = r'\B(?=(\d{3})+(?!\d))'
    return re.sub(pattern, ',', str(num))

# 驼峰命名转下划线命名
def camel_to_snake(name):
    pattern = r'(?<!^)(?=[A-Z])'
    return re.sub(pattern, '_', name).lower()
```

## 性能优化

### 1. 预编译正则表达式

```python
# 不好的做法（每次调用都编译正则）
def bad_extract(text):
    return re.findall(r'\d+', text)

# 好的做法（预编译正则）
NUM_PATTERN = re.compile(r'\d+')
def good_extract(text):
    return NUM_PATTERN.findall(text)
```

### 2. 使用非捕获组

```python
# 不好的做法（不必要的捕获）
re.findall(r'(a|b)c', 'ac bc')  # ['a', 'b']

# 好的做法（非捕获组）
re.findall(r'(?:a|b)c', 'ac bc')  # ['ac', 'bc']
```

### 3. 避免回溯爆炸

```python
# 危险的正则表达式（可能导致回溯爆炸）
dangerous = re.compile(r'(a+)+b')

# 更安全的替代方案
safe = re.compile(r'a+b')
```

## 常见问题与解决方案

### 1. Unicode 字符处理

```python
# 匹配中文字符
re.findall(r'[\u4e00-\u9fa5]+', '你好世界')  # ['你好世界']

# 匹配emoji
re.findall(r'[\U0001F600-\U0001F64F]', 'Hello 😊 World')  # ['😊']
```

### 2. 贪婪与非贪婪匹配

```python
text = '<div>content</div><div>more</div>'

# 贪婪匹配（默认）
re.findall(r'<div>.*</div>', text)  # ['<div>content</div><div>more</div>']

# 非贪婪匹配
re.findall(r'<div>.*?</div>', text)  # ['<div>content</div>', '<div>more</div>']
```

### 3. 多行匹配问题

```python
text = """first line
second line
third line"""

# 匹配以 'line' 结尾的行
re.findall(r'.*line$', text, re.MULTILINE)  # ['first line', 'second line']
```

## 第三方库扩展

### 1. regex 模块

Python 标准库 `re` 的增强版，支持更多特性：

```python
import regex

# 支持递归匹配
pattern = regex.compile(r'\((?:[^()]|(?R))*\)')
pattern.search('(a(b)c)')  # 匹配嵌套括号

# 支持模糊匹配
pattern = regex.compile(r'(?:hello){e<=2}')
pattern.search('hallo')  # 允许最多2个错误
```

### 2. pyparsing

更适合构建复杂解析器的库：

```python
from pyparsing import Word, alphas, nums

# 定义简单解析器
integer = Word(nums)
date = integer + '/' + integer + '/' + integer

date.parseString('12/31/2023')  # 解析日期
```

## 总结

Python 中的正则表达式实现提供了丰富的功能和灵活性：

- **核心功能**：匹配、搜索、替换、分割等基本操作
- **高级特性**：命名捕获组、非捕获组、注释模式、多行模式等
- **实际应用**：数据验证、数据提取、文本处理等场景
- **性能优化**：预编译正则、避免回溯爆炸、使用非捕获组
- **扩展支持**：可通过 `regex` 模块获得更强大的功能

> 提示：对于简单的模式匹配，直接使用 `re` 模块的函数即可；对于需要重复使用的模式，建议预编译正则表达式；对于复杂的文本处理需求，可以考虑结合多个简单的正则表达式或使用专门的解析器库。