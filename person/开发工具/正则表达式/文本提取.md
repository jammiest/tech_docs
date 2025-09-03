# 正则表达式文本提取指南

> 文本提取是正则表达式最强大的应用之一，可以从非结构化文本中精确提取所需信息。本节将详细介绍各种文本提取场景和技巧。

## 基础提取模式

### 1. 提取数字

```regex
# 提取所有整数
\d+

# 提取所有浮点数
\d+\.\d+

# 提取带符号的数字
[-+]?\d+(?:\.\d+)?

# 提取货币金额
\$\d+(?:\.\d{2})?
```

### 2. 提取电子邮件

```regex
# 提取所有电子邮件地址
\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b

# 提取特定域名的邮箱
\b[A-Za-z0-9._%+-]+@(?:gmail|yahoo|outlook)\.com\b

# 提取邮箱用户名部分
(?<=@)[A-Za-z0-9._%+-]+(?=\.[A-Z|a-z]{2,})
```

### 3. 提取URL

```regex
# 提取所有URL
https?://[^\s/$.?#].[^\s]*

# 提取域名部分
(?<=://)[^/]+

# 提取URL路径
https?://[^\s/$.?#]+(/[^\s?#]*)

# 提取查询参数
\?[^#]+

# 提取特定文件扩展名的URL
https?://[^\s/$.?#]+\.[^\s]*(?:pdf|docx?|xlsx?|pptx?)\b
```

## 结构化数据提取

### 1. 日志文件提取

```regex
# Apache/Nginx日志格式提取
^(\S+) (\S+) (\S+) \[([^\]]+)\] "(\S+) (.*?) (\S+)" (\d+) (\d+) "([^"]*)" "([^"]*)"

# 提取IP地址
\b(?:\d{1,3}\.){3}\d{1,3}\b

# 提取时间戳
\[\d{2}/\w{3}/\d{4}:\d{2}:\d{2}:\d{2} [+-]\d{4}\]

# 提取HTTP状态码
\s(\d{3})\s

# 提取用户代理
"([^"]*)"$
```

### 2. CSV/TSV数据提取

```regex
# 提取CSV行
^([^,]*),([^,]*),([^,]*)$

# 处理带引号的CSV字段
"(?:[^"]|"")*"|[^,]*

# 提取TSV字段
([^\t]*)\t([^\t]*)\t([^\t]*)
```

### 3. JSON数据提取

```regex
# 提取JSON键值对（简单版）
"([^"]+)":\s*("[^"]+"|\d+|true|false|null)

# 提取所有字符串值
"([^"]+)"

# 提取数组元素
\[([^\]]+)\]

# 提取嵌套对象
\{([^}]+)\}
```

## 文档内容提取

### 1. HTML内容提取

```regex
# 提取所有链接
<a\s+href=[^"']+["'][^>]*>

# 提取图片src
<img[^"']+["']

# 提取标题文本
<h[1-6][^>]*>(.*?)<\/h[1-6]>

# 提取段落内容
<p[^>]*>(.*?)<\/p>

# 提取div内容（包含属性）
<div[^>]*>(.*?)<\/div>

# 提取表格数据
<td[^>]*>(.*?)<\/td>

# 提取meta标签内容
<meta[^"']+[^"']+["']
```

### 2. Markdown内容提取

```regex
# 提取标题
^#+\s+(.+)$

# 提取链接
\[([^\]]+)\]\(([^)]+)\)

# 提取图片
!\[([^\]]*)\]\(([^)]+)\)

# 提取代码块
```([^`]*)```

# 提取内联代码
`([^`]*)`

# 提取粗体文本
\*\*([^*]+)\*\*|\_\_([^_]+)\_\_

# 提取斜体文本
\*([^*]+)\*|_([^_]+)_
```

### 3. XML内容提取

```regex
# 提取标签内容
<([^>]+)>(.*?)<\/\1>

# 提取属性值
(\w+)="([^"]+)"

# 提取CDATA内容
<!\[CDATA\[(.*?)\]\]>

# 提取注释内容
<!--(.*?)-->
```

## 特定领域提取

### 1. 代码文件提取

```javascript
// 提取函数定义
function\s+([a-zA-Z_$][0-9a-zA-Z_$]*)\s*\([^)]*\)\s*\{?

// 提取变量声明
(?:var|let|const)\s+([a-zA-Z_$][0-9a-zA-Z_$]*)

// 提取字符串字面量
(['"])(?:(?!\1|\\).|\\.)*\1

// 提取注释
\/\/.*$|\/\*[\s\S]*?\*\/

// 提取import语句
import\s+(?:[^'"]+['"]
```

### 2. 配置文件提取

```regex
# 提取INI配置项
^([^=]+)=([^#]*)

# 提取环境变量
^([A-Z_]+)=([^#]*)

# 提取YAML键值对
^([^:]+):\s*(.*)$

# 提取.properties文件内容
^([^=#!]+)=([^#!]*)"
```

### 3. 社交媒体内容提取

```regex
# 提取话题标签
#\w+

# 提取提及用户
@\w+

# 提取表情符号
[\x{1F600}-\x{1F64F}\x{1F300}-\x{1F5FF}\x{1F680}-\x{1F6FF}\x{2600}-\x{26FF}]

# 提取URL缩短服务链接
(?:bit\.ly|t\.co|goo\.gl|tinyurl\.com)/[a-zA-Z0-9]+
```

## 高级提取技巧

### 1. 条件提取

```regex
# 只提取特定上下文中的数字
(?<=Price: )\d+(?:\.\d{2})?

# 提取不在引号内的逗号（用于CSV处理）
,(?=(?:[^"]*"[^"]*")*[^"]*$)

# 提取特定标签属性的内容
<div class="price">(.*?)<\/div>
```

### 2. 分组提取

```regex
# 提取日期各部分
(\d{4})-(\d{2})-(\d{2})

# 提取姓名各部分
([A-Z][a-z]+)\s+([A-Z][a-z]+)

# 提取地址组成部分
(\d+)\s+([^,]+),\s*([^,]+),\s*([A-Z]{2})\s+(\d{5})
```

### 3. 多行提取

```regex
# 提取多行代码注释
\/\*\s*([^*]|(\*+[^*/]))*\*+\/

# 提取多行段落
(?s)<p>(.*?)<\/p>

# 提取表格行
(?m)^\|(.+)\|$
```

## 性能优化建议

1. **使用非贪婪匹配**：避免不必要的回溯
   ```regex
   # 贪婪（可能性能差）
   <div>.*<\/div>
   
   # 非贪婪（性能更好）
   <div>.*?<\/div>
   ```

2. **具体化匹配模式**：减少匹配范围
   ```regex
   # 不好
   .*abc
   
   # 更好
   [^abc]*abc
   ```

3. **避免复杂回溯**：简化正则表达式结构
   ```regex
   # 危险模式
   (a+)+b
   
   # 安全模式
   a+b
   ```

## 各语言实现示例

### JavaScript 实现

```javascript
// 提取所有电子邮件
function extractEmails(text) {
  const regex = /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g;
  return text.match(regex) || [];
}

// 提取HTML链接
function extractLinks(html) {
  const regex = /<a\s+href=[^"']+["'][^>]*>/gi;
  const links = [];
  let match;
  while ((match = regex.exec(html)) !== null) {
    links.push(match[1]);
  }
  return links;
}
```

### Python 实现

```python
import re

def extract_phone_numbers(text):
    pattern = r'\b1[3-9]\d{9}\b'
    return re.findall(pattern, text)

def extract_dates(text):
    pattern = r'\b\d{4}-\d{2}-\d{2}\b'
    return re.findall(pattern, text)

def extract_table_data(html):
    pattern = r'<td[^>]*>(.*?)</td>'
    return re.findall(pattern, html, re.DOTALL)
```

### PHP 实现

```php
function extractUrls($text) {
    preg_match_all('/https?:\/\/[^\s\/$.?#].[^\s]*/', $text, $matches);
    return $matches[0];
}

function extractHashtags($text) {
    preg_match_all('/#\w+/', $text, $matches);
    return $matches[0];
}

function extractMentions($text) {
    preg_match_all('/@\w+/', $text, $matches);
    return $matches[0];
}
```

### Java 实现

```java
import java.util.regex.*;

public class TextExtractor {
    public static List<String> extractEmails(String text) {
        Pattern pattern = Pattern.compile("\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}\\b");
        Matcher matcher = pattern.matcher(text);
        List<String> emails = new ArrayList<>();
        while (matcher.find()) {
            emails.add(matcher.group());
        }
        return emails;
    }
    
    public static List<String> extractPhoneNumbers(String text) {
        Pattern pattern = Pattern.compile("\\b1[3-9]\\d{9}\\b");
        Matcher matcher = pattern.matcher(text);
        List<String> phones = new ArrayList<>();
        while (matcher.find()) {
            phones.add(matcher.group());
        }
        return phones;
    }
}
```

## 实际应用场景

### 1. 网页爬虫数据提取

```python
def extract_product_info(html):
    """提取电商网站商品信息"""
    data = {}
    
    # 提取商品名称
    name_match = re.search(r'<h1[^>]*>(.*?)<\/h1>', html)
    if name_match:
        data['name'] = re.sub(r'<[^>]+>', '', name_match.group(1)).strip()
    
    # 提取价格
    price_match = re.search(r'["']price["'][^>]*>([^<]+)', html)
    if price_match:
        data['price'] = price_match.group(1).strip()
    
    # 提取图片
    image_matches = re.findall(r'<img[^"\']+["\'][^>]*>', html)
    data['images'] = image_matches
    
    return data
```

### 2. 日志分析

```javascript
function parseLogLine(line) {
    const pattern = /^(\S+) (\S+) (\S+) \[([^\]]+)\] "(\S+) (.*?) (\S+)" (\d+) (\d+) "([^"]*)" "([^"]*)"$/;
    const match = pattern.exec(line);
    
    if (match) {
        return {
            ip: match[1],
            timestamp: match[4],
            method: match[5],
            url: match[6],
            protocol: match[7],
            status: match[8],
            size: match[9],
            referer: match[10],
            userAgent: match[11]
        };
    }
    return null;
}
```

### 3. 文档处理

```python
def extract_document_metadata(text):
    """从文档中提取元数据"""
    metadata = {}
    
    # 提取标题
    title_match = re.search(r'<title>(.*?)<\/title>', text, re.IGNORECASE)
    if title_match:
        metadata['title'] = title_match.group(1)
    
    # 提取作者
    author_match = re.search(r'<meta[^"\']+["\']', text)
    if author_match:
        metadata['author'] = author_match.group(1)
    
    # 提取关键词
    keywords_match = re.search(r'<meta[^"\']+["\']', text)
    if keywords_match:
        metadata['keywords'] = keywords_match.group(1).split(',')
    
    return metadata
```

## 总结

正则表达式文本提取的核心要点：

- **基础提取**：数字、邮件、URL等常见内容提取
- **结构化数据**：日志、CSV、JSON等格式解析
- **文档内容**：HTML、Markdown、XML等文档提取
- **特定领域**：代码、配置、社交媒体等内容处理
- **高级技巧**：条件提取、分组提取、多行处理
- **性能优化**：非贪婪匹配、具体化模式、避免回溯

> 提示：对于复杂的文本提取需求，建议结合多个简单的正则表达式分步骤处理，或者使用专门的解析器库。正则表达式虽然强大，但在处理复杂嵌套结构时可能不如专门的解析器高效。