# 正则表达式完全指南

> 正则表达式（Regular Expression，简称 regex）是一种强大的文本处理工具，用于匹配、查找、替换和验证字符串模式。掌握正则表达式可以极大提高文本处理的效率和精确度。

## 目录

1. #基础概念
2. #元字符
3. #字符类
4. #量词
5. #分组和捕获
6. #断言
7. #常用模式
8. #编程语言中的使用
9. #性能优化
10. #工具和资源

## 基础概念

### 什么是正则表达式？

正则表达式是由普通字符（如字母、数字）和特殊字符（称为"元字符"）组成的文本模式，用于描述字符串的匹配规则。

### 基本语法结构

```regex
/pattern/flags
```

- **pattern**: 匹配模式
- **flags**: 修饰符（如 i=忽略大小写，g=全局匹配）

### 简单示例

```regex
# 匹配 "hello"
/hello/

# 匹配数字
/\d/

# 匹配邮箱
/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/
```

## 元字符

### 基本元字符

| 元字符 | 描述                  | 示例             |
|--------|-----------------------|------------------|
| `.`    | 匹配任意单个字符      | `a.c` → "abc"    |
| `^`    | 匹配字符串开始        | `^start`         |
| `$`    | 匹配字符串结束        | `end$`           |
| `\`    | 转义字符              | `\.` → 匹配 "."  |
| `\|`   | 或操作                | `a\|b` → "a"或"b"|

### 字符转义

```regex
# 匹配特殊字符
/\$100/    # 匹配 "$100"
/\(test\)/ # 匹配 "(test)"
/\\/       # 匹配反斜杠
```

## 字符类

### 预定义字符类

| 字符类 | 等价表示     | 描述               |
|--------|--------------|--------------------|
| `\d`   | `[0-9]`      | 数字字符           |
| `\D`   | `[^0-9]`     | 非数字字符         |
| `\w`   | `[a-zA-Z0-9_]`| 单词字符           |
| `\W`   | `[^a-zA-Z0-9_]`| 非单词字符         |
| `\s`   | `[\t\n\r\f\v]`| 空白字符           |
| `\S`   | `[^\t\n\r\f\v]`| 非空白字符         |

### 自定义字符类

```regex
# 匹配元音字母
/[aeiouAEIOU]/

# 匹配十六进制字符
/[0-9a-fA-F]/

# 排除特定字符
/[^0-9]/  # 非数字字符

# 字符范围
/[a-z]/   # 小写字母
/[A-Z]/   # 大写字母
/[0-9]/   # 数字
```

## 量词

### 基本量词

| 量词    | 描述                  | 示例             |
|---------|-----------------------|------------------|
| `*`     | 0次或多次             | `a*` → "", "a", "aa" |
| `+`     | 1次或多次             | `a+` → "a", "aa"     |
| `?`     | 0次或1次              | `a?` → "", "a"       |
| `{n}`   | 恰好n次               | `a{3}` → "aaa"       |
| `{n,}`  | 至少n次               | `a{2,}` → "aa", "aaa"|
| `{n,m}` | n到m次                | `a{2,4}` → "aa", "aaa", "aaaa" |

### 贪婪与非贪婪匹配

```regex
# 贪婪匹配（默认）
/<.*>/   # 匹配整个 "<div>content</div>"

# 非贪婪匹配
/<.*?>/  # 匹配单个 "<div>" 和 "</div>"

# 非贪婪量词
*?      # 非贪婪的 *
+?      # 非贪婪的 +
??      # 非贪婪的 ?
```

## 分组和捕获

### 分组语法

```regex
# 捕获分组
(pattern)      # 捕获匹配的内容

# 非捕获分组
(?:pattern)    # 不捕获匹配的内容

# 命名分组
(?<name>pattern)  # 命名捕获组
```

### 分组示例

```regex
# 匹配日期并捕获各部分
/(\d{4})-(\d{2})-(\d{2})/

# 非捕获分组示例
/(?:https?|ftp):\/\/([^\/]+)/

# 命名分组（Python、JavaScript等支持）
/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
```

### 反向引用

```regex
# 匹配重复单词
/\b(\w+)\s+\1\b/  # "the the", "cat cat"

# 匹配HTML标签
/<(\w+)>.*?<\/\1>/  # "<div>content</div>"
```

## 断言

###  Lookahead 断言

```regex
# 正向先行断言
pattern(?=assert)  # 匹配后面跟着assert的pattern

# 负向先行断言
pattern(?!assert)  # 匹配后面不跟着assert的pattern
```

### Lookbehind 断言

```regex
# 正向后行断言
(?<=assert)pattern  # 匹配前面是assert的pattern

# 负向后行断言
(?<!assert)pattern  # 匹配前面不是assert的pattern
```

### 断言示例

```regex
# 匹配后面跟着px的数字
/\d+(?=px)/  # 匹配 "10px" 中的 "10"

# 匹配不是价格的数字
/\d+(?!\$)/  # 匹配 "10 items" 中的 "10"

# 匹配前面是$的数字
/(?<=\$)\d+/  # 匹配 "$100" 中的 "100"

# 匹配前面不是$的数字
/(?<!\$)\d+/  # 匹配 "price 100" 中的 "100"
```

## 常用模式

### 邮箱验证

```regex
/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/
```

### URL 匹配

```regex
/^(https?|ftp):\/\/[^\s/$.?#].[^\s]*$/
```

### 手机号码

```regex
# 中国手机号
/^1[3-9]\d{9}$/

# 国际手机号（简化）
/^\+?[1-9]\d{1,14}$/
```

### 日期格式

```regex
# YYYY-MM-DD
/^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$/

# DD/MM/YYYY
/^(0[1-9]|[12]\d|3[01])\/(0[1-9]|1[0-2])\/\d{4}$/
```

### 密码强度

```regex
# 至少8字符，包含大小写字母和数字
/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/

# 包含特殊字符
/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&]).{8,}$/
```

## 编程语言中的使用

### JavaScript

```javascript
// 创建正则表达式
const regex = /pattern/flags;
const regexObj = new RegExp('pattern', 'flags');

// 常用方法
const result = regex.test(string);      // 返回布尔值
const matches = string.match(regex);    // 返回匹配数组
const newString = string.replace(regex, replacement);
```

### Python

```python
import re

# 编译正则表达式
pattern = re.compile(r'pattern', flags)

# 常用方法
result = pattern.search(string)    # 搜索匹配
result = pattern.match(string)     # 从开始匹配
matches = pattern.findall(string)  # 所有匹配
new_string = pattern.sub(replacement, string)
```

### Java

```java
import java.util.regex.*;

Pattern pattern = Pattern.compile("pattern");
Matcher matcher = pattern.matcher(input);

while (matcher.find()) {
    System.out.println(matcher.group());
}
```

### PHP

```php
$pattern = '/pattern/';
preg_match($pattern, $string, $matches);
preg_replace($pattern, $replacement, $string);
```

## 性能优化

### 优化技巧

1. **避免回溯爆炸**
   ```regex
   # 不好 - 可能产生大量回溯
   /(a+)+b/

   # 更好
   /a+b/
   ```

2. **使用具体字符类**
   ```regex
   # 不好
   /.*abc/

   # 更好
   /[^abc]*abc/
   ```

3. **避免不必要的捕获**
   ```regex
   # 使用非捕获分组
   /(?:abc|def)/
   ```

4. **使用锚点提高性能**
   ```regex
   # 如果知道模式在开头/结尾
   /^pattern/
   /pattern$/
   ```

### 性能测试工具

- **regex101.com** - 在线测试和调试
- **debuggex.com** - 可视化正则表达式
- **各语言的性能分析工具**

## 工具和资源

### 在线测试工具

- https://regex101.com/ - 功能全面的在线测试器
- https://regexr.com/ - 学习和测试正则表达式
- https://www.debuggex.com/ - 可视化正则表达式

### 学习资源

- https://deerchao.cn/tutorials/regex/regex.htm
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions
- http://www.rexegg.com/ - 高级正则表达式技巧

### 常用参考

| 模式           | 正则表达式                  |
|----------------|---------------------------|
| 整数           | `^-?\d+$`                 |
| 浮点数         | `^-?\d+(\.\d+)?$`         |
| 中文           | `^[\u4e00-\u9fa5]+$`      |
| 邮政编码       | `^\d{6}$`                 |
| IP地址         | `^(\d{1,3}\.){3}\d{1,3}$` |
| HTML标签       | `<\/?[a-z][^>]*>`         |

## 总结

!> **重要提示**：正则表达式虽然强大，但也要注意：
- 复杂的正则表达式可能难以维护
- 某些模式可能导致性能问题
- 考虑使用其他文本处理方式作为补充

正则表达式是每个开发者都应该掌握的重要技能。通过系统学习和实践，你可以：

- ✅ 高效处理文本数据
- ✅ 实现复杂的验证逻辑
- ✅ 提高代码的简洁性和可读性
- ✅ 解决各种字符串处理问题

> 提示：建议从简单的模式开始练习，逐步掌握更复杂的特性。使用在线工具进行测试和调试可以大大提高学习效率。