# 正则表达式在编程语言中的具体实现

> 本章将深入探讨正则表达式在不同编程语言中的具体实现、特性差异和最佳实践。虽然正则表达式的基本概念是通用的，但不同语言的实现细节和特性支持存在显著差异。

## JavaScript 中的正则表达式

### 创建方式

```javascript
// 字面量方式（推荐）
const regex1 = /pattern/flags;

// 构造函数方式
const regex2 = new RegExp('pattern', 'flags');

// 动态构建模式
const dynamicPattern = 'hello';
const dynamicRegex = new RegExp(dynamicPattern, 'gi');
```

### 常用方法

```javascript
const text = "Hello world hello universe";

// test() - 检查是否匹配
const hasHello = /hello/i.test(text); // true

// exec() - 获取匹配详情
const match = /hello/gi.exec(text);
console.log(match[0], match.index); // "Hello", 0

// match() - 字符串方法，获取所有匹配
const allMatches = text.match(/hello/gi); // ["Hello", "hello"]

// replace() - 替换匹配内容
const replaced = text.replace(/hello/gi, 'Hi'); // "Hi world Hi universe"

// search() - 查找匹配位置
const position = text.search(/world/); // 6

// split() - 使用正则表达式分割
const parts = text.split(/\s+/); // ["Hello", "world", "hello", "universe"]
```

### ES6+ 新特性

```javascript
// 命名捕获组
const namedRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const namedMatch = namedRegex.exec('2024-01-15');
console.log(namedMatch.groups.year); // "2024"

// dotAll 模式 (s标志)
const dotAllRegex = /hello.world/s;
console.log(dotAllRegex.test('hello\nworld')); // true

// Unicode属性转义
const unicodeRegex = /\p{Emoji}/u;
console.log(unicodeRegex.test('😊')); // true

// 后行断言
const lookbehindRegex = /(?<=\$)\d+/;
console.log(lookbehindRegex.exec('Price: $100')); // ["100"]
```

## Python 中的正则表达式

### 基本使用

```python
import re

# 编译正则表达式
pattern = re.compile(r'\d{3}-\d{2}-\d{4}')

# 匹配方法
text = "My SSN is 123-45-6789"

# search() - 查找第一个匹配
match = pattern.search(text)
if match:
    print(match.group())  # "123-45-6789"

# findall() - 查找所有匹配
all_matches = pattern.findall(text)  # ["123-45-6789"]

# finditer() - 返回迭代器
for match in pattern.finditer(text):
    print(match.span(), match.group())

# 直接使用模块函数
match = re.search(r'\d{3}-\d{2}-\d{4}', text)
```

### 高级特性

```python
# 命名捕获组
pattern = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})')
match = pattern.search('2024-01-15')
print(match.group('year'))  # "2024"

# 注释模式 (verbose)
complex_pattern = re.compile(r"""
    ^                   # 字符串开始
    (\d{3})             # 区号
    -                   # 分隔符
    (\d{3})             # 前缀
    -                   # 分隔符
    (\d{4})             # 线路号
    $                   # 字符串结束
""", re.VERBOSE)

# 替换回调函数
def mask_ssn(match):
    ssn = match.group()
    return '***-**-' + ssn[-4:]

text = "SSN: 123-45-6789"
masked = re.sub(r'\d{3}-\d{2}-\d{4}', mask_ssn, text)
```

## Java 中的正则表达式

### 基本用法

```java
import java.util.regex.*;

public class RegexExample {
    public static void main(String[] args) {
        String text = "Hello world hello universe";
        Pattern pattern = Pattern.compile("hello", Pattern.CASE_INSENSITIVE);
        Matcher matcher = pattern.matcher(text);
        
        // 查找匹配
        while (matcher.find()) {
            System.out.println("Found: " + matcher.group() + " at " + matcher.start());
        }
        
        // 替换
        String replaced = matcher.replaceAll("Hi");
        System.out.println(replaced); // "Hi world Hi universe"
    }
}
```

### 高级特性

```java
// 命名捕获组 (Java 7+)
Pattern pattern = Pattern.compile("(?<year>\\d{4})-(?<month>\\d{2})-(?<day>\\d{2})");
Matcher matcher = pattern.matcher("2024-01-15");
if (matcher.find()) {
    System.out.println(matcher.group("year")); // "2024"
}

// 区域匹配
matcher.region(10, 20); // 只在指定区域匹配

// 使用Scanner进行正则表达式分割
Scanner scanner = new Scanner("apple,banana,cherry");
scanner.useDelimiter(",");
while (scanner.hasNext()) {
    System.out.println(scanner.next());
}
```

## PHP 中的正则表达式

### PCRE 函数

```php
<?php
$text = "Hello world hello universe";

// preg_match - 第一个匹配
if (preg_match('/hello/i', $text, $matches)) {
    echo "Found: " . $matches[0];
}

// preg_match_all - 所有匹配
preg_match_all('/hello/i', $text, $matches);
print_r($matches[0]); // ["Hello", "hello"]

// preg_replace - 替换
$replaced = preg_replace('/hello/i', 'Hi', $text);
echo $replaced; // "Hi world Hi universe"

// preg_split - 分割
$parts = preg_split('/\s+/', $text);
print_r($parts); // ["Hello", "world", "hello", "universe"]

// 命名捕获组
preg_match('/(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})/', '2024-01-15', $matches);
echo $matches['year']; // "2024"
?>
```

## C# 中的正则表达式

### .NET 实现

```csharp
using System;
using System.Text.RegularExpressions;

class Program
{
    static void Main()
    {
        string text = "Hello world hello universe";
        
        // 基本匹配
        Match match = Regex.Match(text, @"hello", RegexOptions.IgnoreCase);
        if (match.Success)
        {
            Console.WriteLine($"Found: {match.Value} at {match.Index}");
        }
        
        // 所有匹配
        MatchCollection matches = Regex.Matches(text, @"hello", RegexOptions.IgnoreCase);
        foreach (Match m in matches)
        {
            Console.WriteLine(m.Value);
        }
        
        // 替换
        string replaced = Regex.Replace(text, @"hello", "Hi", RegexOptions.IgnoreCase);
        Console.WriteLine(replaced);
    }
}
```

### .NET 特有特性

```csharp
// 平衡组（匹配嵌套结构）
string pattern = @"(?<open>\()(?<content-open>)*?(?<close-open>\))+";
string text = "(a(b)c)";
Match match = Regex.Match(text, pattern);

// 编译正则表达式（提高性能）
Regex compiledRegex = new Regex(@"\d+", RegexOptions.Compiled);

// 超时设置（防止DoS攻击）
var timeout = TimeSpan.FromMilliseconds(100);
var regex = new Regex(@"^(a+)+$", RegexOptions.None, timeout);
```

## 语言特性对比

### 支持程度比较

| 特性 | JavaScript | Python | Java | PHP | C# |
|------|-----------|---------|------|-----|----|
| 命名捕获组 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 后行断言 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 递归匹配 | ❌ | ❌ | ❌ | ✅ | ✅ |
| 条件表达式 | ❌ | ❌ | ❌ | ✅ | ✅ |
| 注释模式 | ✅ | ✅ | ❌ | ✅ | ✅ |
| Unicode属性 | ✅ | ✅ | ✅ | ✅ | ✅ |

### 性能考虑

```javascript
// JavaScript性能优化
// 编译一次，多次使用
const regex = /pattern/gi;
function processText(text) {
    return regex.test(text);
}

// 避免在循环中创建正则表达式
// 不好
for (let i = 0; i < 1000; i++) {
    const regex = new RegExp('pattern' + i); // 每次创建新对象
}

// 好
const regexes = [];
for (let i = 0; i < 1000; i++) {
    regexes.push(new RegExp('pattern' + i));
}
```

## 跨语言最佳实践

### 通用模式

```javascript
// 邮箱验证（跨语言兼容）
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;

// URL验证
const urlRegex = /^(https?|ftp):\/\/[^\s/$.?#].[^\s]*$/;

// 手机号验证（中国）
const phoneRegex = /^1[3-9]\d{9}$/;
```

### 错误处理

```python
# Python中的错误处理
import re

try:
    pattern = re.compile(r'*invalid*')  # 无效模式
    pattern.search('test')
except re.error as e:
    print(f"Regex error: {e}")
```

```javascript
// JavaScript中的错误处理
function safeRegex(pattern, flags) {
    try {
        return new RegExp(pattern, flags);
    } catch (error) {
        console.warn('Invalid regex pattern:', error.message);
        return null;
    }
}

const regex = safeRegex('*invalid*');
if (regex) {
    // 使用regex
}
```

### 性能测试工具

```python
# Python性能测试
import re
import time

def test_regex_performance(pattern, test_string, iterations=1000):
    regex = re.compile(pattern)
    start_time = time.time()
    
    for _ in range(iterations):
        regex.search(test_string)
    
    duration = time.time() - start_time
    return duration / iterations

# 测试不同模式
patterns = [
    r'(a+)+b',    # 危险模式
    r'a+b',       # 安全模式
    r'(?:a+)+b'   # 优化模式
]

for pattern in patterns:
    avg_time = test_regex_performance(pattern, 'aaaaaaaaac')
    print(f"{pattern}: {avg_time:.6f}s per iteration")
```

## 语言特定技巧

### JavaScript 技巧

```javascript
// 使用标签函数进行模板化正则表达式
function regexTemplate(strings, ...values) {
    const pattern = strings.reduce((result, string, i) => {
        return result + string + (values[i] || '');
    }, '');
    return new RegExp(pattern);
}

const digitPattern = regexTemplate`\d{${3}}-\d{${2}}-\d{${4}}`;
console.log(digitPattern.test('123-45-6789')); // true
```

### Python 技巧

```python
# 使用re.sub的高级替换
def complex_replacement(match):
    # 基于匹配内容进行复杂替换逻辑
    value = match.group()
    if value.isdigit():
        return f"Number: {value}"
    else:
        return f"Text: {value}"

text = "123 abc 456 def"
result = re.sub(r'\w+', complex_replacement, text)
print(result)  # "Number: 123 Text: abc Number: 456 Text: def"
```

### Java 技巧

```java
// 使用Pattern.quote进行安全匹配
String userInput = ".*+"; // 可能包含正则元字符
String safePattern = Pattern.quote(userInput);
Pattern pattern = Pattern.compile(safePattern);
Matcher matcher = pattern.matcher("Some .*+ text");
```

## 总结

不同编程语言中的正则表达式实现各有特点：

- **JavaScript**: 强大的ES6+特性，良好的浏览器支持
- **Python**: 清晰的API，优秀的字符串处理能力  
- **Java**: 企业级应用，强大的性能优化选项
- **PHP**: PCRE支持，丰富的内置函数
- **C#**: .NET特有特性，如平衡组和编译优化

!> **重要建议**：在选择正则表达式实现时，考虑：
1. 目标环境的特性支持
2. 性能要求
3. 团队熟悉程度
4. 跨平台兼容性需求

> 提示：对于复杂的文本处理需求，可以考虑使用专门的解析器生成器（如ANTLR），或者结合多种文本处理技术来实现最佳效果。