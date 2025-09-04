# 正则表达式字符串替换指南

> 字符串替换是正则表达式最常用的功能之一，可以实现复杂的文本转换和格式化操作。本节将详细介绍各种替换场景和技巧。

## 基础替换语法

### 1. 简单替换

```regex
# 将所有数字替换为 #
\#\d+\# → "###"

# 将多个空格替换为单个空格
\s+ → " "

# 删除所有标点符号
[[:punct:]] → ""
```

### 2. 分组引用

```regex
# 重排日期格式 (YYYY-MM-DD → DD/MM/YYYY)
(\d{4})-(\d{2})-(\d{2}) → \3/\2/\1

# 格式化电话号码
(\d{3})(\d{3})(\d{4}) → (\1) \2-\3

# 交换名字和姓氏
(\w+)\s+(\w+) → \2, \1
```

### 3. 条件替换

```regex
# 只在特定上下文中替换
(?<=Price: )\d+ → "###"

# 替换不在引号内的逗号
,(?=(?:[^"]*"[^"]*")*[^"]*$) → ";"
```

## 常见替换场景

### 1. 数据脱敏

```regex
# 隐藏信用卡号中间部分
(\d{4})\d{8}(\d{4}) → \1********\2

# 隐藏邮箱用户名部分
(\w)[^@]*@ → \1***@

# 隐藏手机号中间四位
(\d{3})\d{4}(\d{4}) → \1****\2
```

### 2. 格式标准化

```regex
# 统一日期格式
(\d{2})/(\d{2})/(\d{4}) → \3-\1-\2

# 统一电话号码格式
(\d{3})[ -]?(\d{3})[ -]?(\d{4}) → (\1) \2-\3

# 统一货币格式
\$(\d+)(?:\.(\d{2}))? → \1.\2 USD
```

### 3. 代码重构

```regex
# 重命名变量
\b(oldVar)\b → newVar

# 函数参数重排
function\s+(\w+)\((\w+),\s*(\w+)\) → function \1(\3, \2)

# 添加类型提示
function\s+(\w+)\((.*?)\) → function \1($2): void
```

### 4. 文档格式化

```regex
# Markdown标题升级
^#\s+(.*)$ → ## \1

# 统一引号风格
"([^"]*)" → '\1'

# 列表项重新编号
^\d+\. → 1.
```

## 高级替换技巧

### 1. 回调函数替换

```javascript
// JavaScript示例
text.replace(/\d+/g, match => parseInt(match) * 2);

// Python示例
re.sub(r'\d+', lambda m: str(int(m.group()) * 2), text)

// PHP示例
preg_replace_callback('/\d+/', function($matches) {
    return $matches[0] * 2;
}, $text);
```

### 2. 条件替换

```regex
# 只替换特定前缀的数字
(?<=\$)\d+ → ###

# 替换不在HTML标签内的内容
(?<=>)([^<]+)(?=<) → UPPERCASE
```

### 3. 递归替换

```python
# 递归移除嵌套括号
import re

def remove_parentheses(text):
    while re.search(r'\([^()]*\)', text):
        text = re.sub(r'\([^()]*\)', '', text)
    return text
```

## 各语言实现示例

### JavaScript 实现

```javascript
// 简单替换
const text1 = "Hello World".replace(/World/, "JavaScript");
console.log(text1); // "Hello JavaScript"

// 分组引用
const text2 = "2023-05-15".replace(/(\d{4})-(\d{2})-(\d{2})/, "$2/$3/$1");
console.log(text2); // "05/15/2023"

// 回调函数替换
const text3 = "1 2 3 4".replace(/\d+/g, match => parseInt(match) * 2);
console.log(text3); // "2 4 6 8"

// 全局替换
const text4 = "a,b,c".replace(/,/g, ";");
console.log(text4); // "a;b;c"
```

### Python 实现

```python
import re

# 简单替换
text1 = re.sub(r'World', 'Python', 'Hello World')
print(text1)  # "Hello Python"

# 分组引用
text2 = re.sub(r'(\d{4})-(\d{2})-(\d{2})', r'\2/\3/\1', '2023-05-15')
print(text2)  # "05/15/2023"

# 回调函数替换
text3 = re.sub(r'\d+', lambda m: str(int(m.group()) * 2), '1 2 3 4')
print(text3)  # "2 4 6 8"

# 全局替换
text4 = re.sub(r',', ';', 'a,b,c')
print(text4)  # "a;b;c"
```

### PHP 实现

```php
// 简单替换
$text1 = preg_replace('/World/', 'PHP', 'Hello World');
echo $text1; // "Hello PHP"

// 分组引用
$text2 = preg_replace('/(\d{4})-(\d{2})-(\d{2})/', '$2/$3/$1', '2023-05-15');
echo $text2; // "05/15/2023"

// 回调函数替换
$text3 = preg_replace_callback('/\d+/', function($matches) {
    return $matches[0] * 2;
}, '1 2 3 4');
echo $text3; // "2 4 6 8"

// 全局替换
$text4 = preg_replace('/,/', ';', 'a,b,c');
echo $text4; // "a;b;c"
```

### Java 实现

```java
import java.util.regex.*;

public class RegexReplace {
    public static void main(String[] args) {
        // 简单替换
        String text1 = "Hello World".replaceAll("World", "Java");
        System.out.println(text1); // "Hello Java"
        
        // 分组引用
        String text2 = "2023-05-15".replaceAll("(\\d{4})-(\\d{2})-(\\d{2})", "$2/$3/$1");
        System.out.println(text2); // "05/15/2023"
        
        // 复杂的回调替换需要自己实现
        String text3 = "1 2 3 4";
        StringBuffer sb = new StringBuffer();
        Matcher m = Pattern.compile("\\d+").matcher(text3);
        while (m.find()) {
            m.appendReplacement(sb, String.valueOf(Integer.parseInt(m.group()) * 2));
        }
        m.appendTail(sb);
        System.out.println(sb.toString()); // "2 4 6 8"
        
        // 全局替换
        String text4 = "a,b,c".replaceAll(",", ";");
        System.out.println(text4); // "a;b;c"
    }
}
```

## 实际应用示例

### 1. 数据清洗

```python
import re

def clean_data(text):
    # 移除多余空格
    text = re.sub(r'\s+', ' ', text)
    # 标准化日期格式
    text = re.sub(r'(\d{2})/(\d{2})/(\d{4})', r'\3-\1-\2', text)
    # 隐藏敏感信息
    text = re.sub(r'\b\d{4}-\d{4}-\d{4}-(\d{4})\b', r'****-****-****-\1', text)
    # 统一货币格式
    text = re.sub(r'\$\s*(\d+(?:\.\d{2})?)', r'\1 USD', text)
    return text
```

### 2. 代码重构

```javascript
// 将旧式函数声明转换为箭头函数
function convertToArrowFunctions(code) {
    return code.replace(
        /function\s+(\w+)\s*\(([^)]*)\)\s*\{([^}]*)\}/g,
        'const $1 = ($2) => {$3}'
    );
}

// 示例使用
const oldCode = `
function add(a, b) {
    return a + b;
}
`;
const newCode = convertToArrowFunctions(oldCode);
console.log(newCode);
// 输出:
// const add = (a, b) => {
//     return a + b;
// }
```

### 3. 文档转换

```php
// 将Markdown链接转换为HTML
function markdownToHtml($markdown) {
    // 转换标题
    $html = preg_replace('/^# (.*$)/m', '<h1>$1</h1>', $markdown);
    $html = preg_replace('/^## (.*$)/m', '<h2>$1</h2>', $html);
    
    // 转换链接
    $html = preg_replace('/\[([^\]]+)\]\(([^)]+)\)/', '<a href="$2">$1</a>', $html);
    
    // 转换粗体
    $html = preg_replace('/\*\*([^*]+)\*\*/', '<strong>$1</strong>', $html);
    
    return $html;
}

// 示例使用
$markdown = "# Title\n\nThis is **bold** and https://example.com";
echo markdownToHtml($markdown);
```

### 4. 日志格式化

```java
public class LogFormatter {
    public static String formatLogEntry(String logLine) {
        // 提取并重排日志信息
        String formatted = logLine.replaceAll(
            "^([^ ]+) ([^ ]+) ([^ ]+) \\[([^\\]]+)\\] \"([^\"]+)\" (\\d+) (\\d+) \"([^\"]+)\" \"([^\"]+)\"$",
            "IP: $1 | Time: $4 | Method: $5 | Status: $6 | Size: $7 | Referer: $8 | Agent: $9"
        );
        
        // 缩短过长的用户代理
        formatted = formatted.replaceAll(
            "Agent: (.{50}).*", 
            "Agent: $1..."
        );
        
        return formatted;
    }
    
    public static void main(String[] args) {
        String log = "127.0.0.1 - - [10/Oct/2023:13:55:36 +0000] \"GET /index.html HTTP/1.1\" 200 2326 \"-\" \"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36\"";
        System.out.println(formatLogEntry(log));
    }
}
```

## 性能优化建议

1. **预编译正则表达式**：对于重复使用的模式
   ```python
   # Python示例
   import re
   pattern = re.compile(r'\d+')
   text = pattern.sub('#', '123abc456')
   ```

2. **避免过度使用回溯**：简化正则表达式结构
   ```regex
   # 不好
   (.*?)abc
   
   # 更好
   [^abc]*abc
   ```

3. **使用非捕获分组**：当不需要捕获内容时
   ```regex
   # 不好
   (a|b)c
   
   # 更好
   (?:a|b)c
   ```

4. **具体化匹配范围**：减少不必要的匹配尝试
   ```regex
   # 不好
   .*abc
   
   # 更好
   [^abc]*abc
   ```

## 常见问题与解决方案

### 1. 特殊字符处理

```regex
# 替换点号（需要转义）
\. → "dot"

# 替换反斜杠（需要双重转义）
\\\\ → "/"

# 替换美元符号（在替换字符串中有特殊含义）
\$\d+ → "###"
```

### 2. 多行替换

```regex
# 替换每行的开头
(?m)^ → // 

# 替换多行注释
/\*.*?\*/ → "" (使用单行模式/s)
```

### 3. 复杂替换逻辑

```python
# 使用回调函数处理复杂逻辑
import re

def complex_replace(match):
    value = int(match.group())
    if value > 100:
        return "HIGH"
    else:
        return "LOW"

text = "50 101 75 200"
result = re.sub(r'\d+', complex_replace, text)
print(result)  # "LOW HIGH LOW HIGH"
```

## 总结

正则表达式字符串替换是强大的文本处理工具：

- **基础替换**：简单文本替换和格式转换
- **分组引用**：重排和重组匹配内容
- **高级技巧**：回调函数、条件替换、递归处理
- **实际应用**：数据清洗、代码重构、文档转换
- **性能优化**：预编译模式、简化表达式、避免回溯

> 提示：对于复杂的替换需求，建议分步骤处理或结合其他字符串操作方法。正则表达式虽然强大，但在处理复杂嵌套结构或需要上下文感知的替换时可能不是最佳选择。