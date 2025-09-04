# Java 中的正则表达式实现

> Java 通过 `java.util.regex` 包提供了全面的正则表达式支持，包括模式匹配、分组捕获和替换操作。本节将详细介绍 Java 中正则表达式的使用方法和特性。

## 基本用法

### 导入相关类

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;
```

### 核心类

| 类 | 描述 |
|----|------|
| `Pattern` | 编译后的正则表达式模式 |
| `Matcher` | 执行匹配操作的引擎 |
| `PatternSyntaxException` | 正则表达式语法异常 |

## 创建和使用正则表达式

### 1. 编译正则表达式

```java
Pattern pattern = Pattern.compile("\\d+");  // 匹配一个或多个数字
```

### 2. 进行匹配

```java
Matcher matcher = pattern.matcher("123abc");
boolean matches = matcher.matches();  // false (不完全匹配)
boolean find = matcher.find();        // true (找到匹配)
```

### 3. 常用方法

| 方法 | 描述 | 示例 |
|------|------|------|
| `Pattern.matches()` | 快速匹配 | `Pattern.matches("\\d+", "123")` |
| `Matcher.matches()` | 完全匹配 | `matcher.matches()` |
| `Matcher.find()` | 查找下一个匹配 | `matcher.find()` |
| `Matcher.group()` | 获取匹配内容 | `matcher.group()` |
| `Matcher.start()` | 匹配开始位置 | `matcher.start()` |
| `Matcher.end()` | 匹配结束位置 | `matcher.end()` |

## 正则表达式标志

Java 支持以下正则表达式标志：

| 标志 | 描述 | 对应常量 |
|------|------|----------|
| `i` | 忽略大小写 | `Pattern.CASE_INSENSITIVE` |
| `m` | 多行模式 | `Pattern.MULTILINE` |
| `s` | dotAll模式 | `Pattern.DOTALL` |
| `x` | 注释模式 | `Pattern.COMMENTS` |
| `u` | Unicode模式 | `Pattern.UNICODE_CASE` |

### 使用标志的两种方式

```java
// 方式1：在模式字符串中嵌入标志
Pattern pattern = Pattern.compile("(?i)abc");  // 忽略大小写

// 方式2：使用常量组合
Pattern pattern = Pattern.compile("abc", 
    Pattern.CASE_INSENSITIVE | Pattern.MULTILINE);
```

## 分组与捕获

### 1. 基本分组

```java
Pattern pattern = Pattern.compile("(\\d{3})-(\\d{4})");
Matcher matcher = pattern.matcher("123-4567");
if (matcher.matches()) {
    String areaCode = matcher.group(1);  // "123"
    String number = matcher.group(2);     // "4567"
}
```

### 2. 命名捕获组（Java 7+）

```java
Pattern pattern = Pattern.compile("(?<area>\\d{3})-(?<number>\\d{4})");
Matcher matcher = pattern.matcher("123-4567");
if (matcher.matches()) {
    String area = matcher.group("area");    // "123"
    String number = matcher.group("number"); // "4567"
}
```

### 3. 非捕获组

```java
Pattern pattern = Pattern.compile("(?:\\d{3})-(\\d{4})");
Matcher matcher = pattern.matcher("123-4567");
if (matcher.matches()) {
    String number = matcher.group(1);  // "4567"
    // group(0) 是整个匹配 "123-4567"
}
```

## 边界匹配

Java 支持以下边界匹配符：

| 边界符 | 描述 |
|--------|------|
| `^` | 行/字符串开头 |
| `$` | 行/字符串结尾 |
| `\b` | 单词边界 |
| `\B` | 非单词边界 |
| `\A` | 输入开头 |
| `\z` | 输入结尾 |
| `\Z` | 输入结尾（忽略最后的终止符） |

### 边界匹配示例

```java
// 匹配独立的单词 "the"
Pattern pattern = Pattern.compile("\\bthe\\b");
Matcher matcher = pattern.matcher("the theme");
matcher.find();  // true (匹配第一个 "the")
matcher.find();  // false (不匹配 "theme" 中的 "the")
```

## 量词与贪婪模式

### 基本量词

| 量词 | 描述 |
|------|------|
| `*` | 零次或多次 |
| `+` | 一次或多次 |
| `?` | 零次或一次 |
| `{n}` | 恰好n次 |
| `{n,}` | 至少n次 |
| `{n,m}` | n到m次 |

### 贪婪与非贪婪

```java
// 贪婪匹配（默认）
Pattern greedy = Pattern.compile("a.*b");
Matcher gm = greedy.matcher("aabbab");
gm.find();  // 匹配整个 "aabbab"

// 非贪婪匹配
Pattern reluctant = Pattern.compile("a.*?b");
Matcher rm = reluctant.matcher("aabbab");
rm.find();  // 匹配 "aab"
rm.find();  // 匹配 "ab"
```

### 占有量词（Possessive Quantifiers）

```java
// 占有量词（不回溯）
Pattern possessive = Pattern.compile("a*+b");
Matcher pm = possessive.matcher("aaaa");
pm.matches();  // false (不会回溯释放a来匹配b)
```

## 实际应用示例

### 1. 数据验证

```java
// 验证邮箱地址
public static boolean isValidEmail(String email) {
    return Pattern.matches("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$", email);
}

// 验证手机号码（中国）
public static boolean isValidChinesePhone(String phone) {
    return Pattern.matches("^1[3-9]\\d{9}$", phone);
}

// 验证强密码（至少8字符，包含大小写字母和数字）
public static boolean isStrongPassword(String password) {
    return Pattern.matches("^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).{8,}$", password);
}
```

### 2. 数据提取

```java
// 从文本中提取所有URL
public static List<String> extractUrls(String text) {
    List<String> urls = new ArrayList<>();
    Pattern pattern = Pattern.compile("https?://[^\\s/$.?#].[^\\s]*");
    Matcher matcher = pattern.matcher(text);
    while (matcher.find()) {
        urls.add(matcher.group());
    }
    return urls;
}

// 从HTML中提取所有图片src
public static List<String> extractImageSrcs(String html) {
    List<String> srcs = new ArrayList<>();
    Pattern pattern = Pattern.compile("<img[^>]+src=\"([^\"]+)\"");
    Matcher matcher = pattern.matcher(html);
    while (matcher.find()) {
        srcs.add(matcher.group(1));
    }
    return srcs;
}
```

### 3. 文本处理

```java
// 移除HTML标签
public static String removeHtmlTags(String html) {
    return html.replaceAll("<[^>]+>", "");
}

// 格式化数字（添加千位分隔符）
public static String formatNumber(long number) {
    return String.valueOf(number).replaceAll("\\B(?=(\\d{3})+(?!\\d))", ",");
}

// 驼峰命名转下划线命名
public static String camelToSnake(String name) {
    return name.replaceAll("([a-z])([A-Z])", "$1_$2").toLowerCase();
}
```

## 高级特性

### 1. 区域匹配（Region Matching）

```java
Pattern pattern = Pattern.compile("\\d+");
Matcher matcher = pattern.matcher("123abc456def");
matcher.region(3, 9);  // 只在 "abc456" 中匹配
matcher.find();        // true (匹配 "456")
```

### 2. 重置匹配器

```java
Pattern pattern = Pattern.compile("\\d+");
Matcher matcher = pattern.matcher("123 abc 456");
matcher.find();  // true (匹配 "123")
matcher.reset();  // 重置匹配器
matcher.find();  // true (再次匹配 "123")
```

### 3. 使用String的split方法

```java
String[] parts = "a1b2c3".split("\\d");  // ["a", "b", "c"]
```

## 性能优化

### 1. 预编译正则表达式

```java
// 不好的做法（每次调用都编译正则）
public static boolean badCheck(String input) {
    return input.matches("\\d+");
}

// 好的做法（预编译正则）
private static final Pattern NUMBER_PATTERN = Pattern.compile("\\d+");
public static boolean goodCheck(String input) {
    return NUMBER_PATTERN.matcher(input).matches();
}
```

### 2. 避免回溯爆炸

```java
// 危险的正则表达式（可能导致回溯爆炸）
Pattern dangerous = Pattern.compile("(a+)+b");

// 更安全的替代方案
Pattern safe = Pattern.compile("a+b");
```

### 3. 使用具体字符类

```java
// 不好
Pattern bad = Pattern.compile(".*abc");

// 更好
Pattern good = Pattern.compile("[^abc]*abc");
```

## 常见问题与解决方案

### 1. 转义字符处理

```java
// 匹配点号（需要双重转义）
Pattern dotPattern = Pattern.compile("\\.");

// 匹配反斜杠（需要四重转义）
Pattern backslashPattern = Pattern.compile("\\\\");
```

### 2. Unicode字符处理

```java
// 匹配中文字符
Pattern chinesePattern = Pattern.compile("[\\u4e00-\\u9fa5]+");

// 匹配emoji（Java 7+）
Pattern emojiPattern = Pattern.compile("[\\x{1F600}-\\x{1F64F}]");
```

### 3. 多行匹配问题

```java
String text = "first line\nsecond line\nthird line";

// 不使用多行模式
Pattern.compile("^.*$").matcher(text).find();  // false

// 使用多行模式
Pattern.compile("^.*$", Pattern.MULTILINE).matcher(text).find();  // true
```

## Java 8+ 增强功能

### 1. 命名捕获组支持增强

```java
// Java 7+ 支持命名捕获组
Pattern pattern = Pattern.compile("(?<hour>\\d{2}):(?<minute>\\d{2})");
Matcher matcher = pattern.matcher("12:30");
if (matcher.matches()) {
    String hour = matcher.group("hour");    // "12"
    String minute = matcher.group("minute"); // "30"
}
```

### 2. Lambda表达式结合正则

```java
// 使用流处理所有匹配结果
Pattern pattern = Pattern.compile("\\d+");
Matcher matcher = pattern.matcher("a1b2c3");
List<String> numbers = matcher.results()  // Java 9+
    .map(MatchResult::group)
    .collect(Collectors.toList());  // ["1", "2", "3"]
```

## 总结

Java 中的正则表达式实现提供了强大的文本处理能力：

- **核心类**：`Pattern` 和 `Matcher` 是主要操作类
- **分组捕获**：支持编号分组和命名分组（Java 7+）
- **边界匹配**：丰富的边界匹配符支持
- **量词控制**：贪婪、非贪婪和占有量词
- **实际应用**：数据验证、提取和文本处理
- **性能优化**：预编译正则、避免回溯爆炸

> 提示：对于复杂的文本处理需求，可以考虑结合多个简单的正则表达式分步骤处理，或者使用专门的解析器库（如 ANTLR）。Java 的正则表达式功能虽然强大，但在处理复杂嵌套结构时可能不如专门的解析器高效。