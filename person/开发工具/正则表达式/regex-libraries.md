# 正则表达式库大全

> 正则表达式库为开发者提供了增强的功能、更好的性能和更方便的API。本节将详细介绍各种编程语言中的优秀正则表达式库。

## JavaScript 正则库

### 1. XRegExp - 功能增强库

**官网**: https://xregexp.com/

#### 安装方式：
```bash
npm install xregexp
```

#### 核心功能：
```javascript
import XRegExp from 'xregexp';

// 1. 命名捕获组
const dateRegex = XRegExp('(?<year>\\d{4})-(?<month>\\d{2})');
const match = XRegExp.exec('2023-05', dateRegex);
console.log(match.year); // "2023"

// 2. Unicode支持
const unicodeRegex = XRegExp('^\\p{L}+$', 'u');
console.log(unicodeRegex.test('中文')); // true

// 3. 模式修饰符
const regex = XRegExp('^abc', 'i'); // 忽略大小写

// 4. 实用方法
XRegExp.forEach('abc123', /\d/, (match) => {
    console.log(match[0]);
});
```

#### 特色功能：
- 支持Unicode类别（\p{L}、\p{N}等）
- 扩展的语法和修饰符
- 强大的工具方法
- 良好的浏览器兼容性

### 2. regexp-tree - 解析和优化

**官网**: https://github.com/DmitrySoshnikov/regexp-tree

#### 安装方式：
```bash
npm install regexp-tree
```

#### 核心功能：
```javascript
import { parse, generate, optimize } from 'regexp-tree';

// 1. 解析正则表达式
const ast = parse('/[a-z]+/i');
console.log(ast);

// 2. 生成正则表达式
const optimized = generate(ast);
console.log(optimized);

// 3. 优化正则表达式
const original = /[a-zA-Z]/;
const optimizedRe = optimize(original);
// 优化为: /[a-z]/i
```

### 3. safe-regex - 安全检测

**官网**: https://github.com/davisjam/safe-regex

#### 安装方式：
```bash
npm install safe-regex
```

#### 核心功能：
```javascript
const safe = require('safe-regex');

// 检测危险的正则表达式
console.log(safe('(a+)+b')); // false
console.log(safe('a+b'));     // true

// 自定义复杂度阈值
console.log(safe('(a|b)*c', { limit: 25 }));
```

## Python 正则库

### 1. regex - 增强版re模块

**官网**: https://pypi.org/project/regex/

#### 安装方式：
```bash
pip install regex
```

#### 核心功能：
```python
import regex

# 1. 递归匹配
pattern = regex.compile(r'\((?:[^()]|(?R))*\)')
matches = pattern.findall('(a(b)c)')

# 2. 模糊匹配
fuzzy_pattern = regex.compile(r'(?:abc){e<=1}')
print(fuzzy_pattern.search('abx'))  # 匹配，允许1个错误

# 3. 命名捕获组
result = regex.search(r'(?P<year>\d{4})', '2023-05')
print(result.group('year'))  # "2023"

# 4. 增强的Unicode支持
unicode_pattern = regex.compile(r'\p{Script=Han}+')
print(unicode_pattern.findall('中文测试'))
```

#### 特色功能：
- 完全兼容标准re模块
- 支持递归匹配(?R)
- 模糊匹配（允许错误）
- 更好的Unicode支持

### 2. pregex - 可读性优先

**官网**: https://pypi.org/project/pregex/

#### 安装方式：
```bash
pip install pregex
```

#### 核心功能：
```python
from pregex import *

# 构建可读的正则表达式
pattern = (
    Group("http", "s", quantifier="?") + "://" +
    Group(Word, quantifier=1, name="domain") +
    Optional("/" + Group(Any, quantifier="*", name="path"))
)

print(pattern)  # 生成清晰的正则表达式
```

## Java 正则库

### 1. jregex - 高性能库

**官网**: https://github.com/eropple/jregex

#### Maven依赖：
```xml
<dependency>
    <groupId>net.oneandone</groupId>
    <artifactId>jregex</artifactId>
    <version>1.1.1</version>
</dependency>
```

#### 核心功能：
```java
import jregex.Pattern;
import jregex.Matcher;

// 1. 命名捕获组
Pattern pattern = new Pattern("(?<year>\\d{4})");
Matcher matcher = pattern.matcher("2023-05");
if (matcher.find()) {
    System.out.println(matcher.group("year"));
}

// 2. 高性能匹配
Pattern compiled = Pattern.compile("\\d+");
Matcher matcher = compiled.matcher("abc123def");
while (matcher.find()) {
    System.out.println(matcher.group());
}
```

### 2. RE2J - Google RE2的Java实现

**官网**: https://github.com/google/re2j

#### Maven依赖：
```xml
<dependency>
    <groupId>com.google.re2j</groupId>
    <artifactId>re2j</artifactId>
    <version>1.7</version>
</dependency>
```

#### 核心功能：
```java
import com.google.re2j.Pattern;

// 1. 保证线性时间匹配（无回溯爆炸）
Pattern pattern = Pattern.compile("(a+)+b");
Matcher matcher = pattern.matcher("aaaaaaaaac");
// 安全匹配，不会导致性能问题

// 2. 线程安全
// RE2J模式是线程安全的，可以共享使用
```

## PHP 正则库

### 1. T-Regx - 现代正则库

**官网**: https://github.com/T-Regx/T-Regx

#### 安装方式：
```bash
composer require rawr/t-regx
```

#### 核心功能：
```php
<?php
use TRegx\CleanRegex\Pattern;

// 1. 链式调用和异常处理
$pattern = Pattern::of('\d+');
$matches = $pattern->match('abc123def')->all();

// 2. 安全的API设计
try {
    $pattern = Pattern::of('invalid [pattern');
} catch (MalformedPatternException $e) {
    // 优雅处理错误
}

// 3. 现代API
$result = Pattern::of('(?<year>\d{4})')
    ->match('2023-05')
    ->group('year')
    ->first();
```

### 2. RegexGuard - 安全防护

**官网**: https://github.com/fojte/regex-guard

#### 核心功能：
```php
<?php
use RegexGuard\RegexGuard;

// 防止正则表达式攻击
$guard = new RegexGuard();
$guard->setTimeout(100); // 100ms超时

try {
    $result = $guard->execute(function() {
        return preg_match('/(a+)+b/', 'aaaaaaaaac');
    });
} catch (RegexTimeoutException $e) {
    // 处理超时
}
```

## .NET 正则库

### 1. NRex - .NET增强库

**官网**: https://github.com/nreco/nregex

#### NuGet安装：
```bash
Install-Package NReco.Regex
```

#### 核心功能：
```csharp
using NReco.Regex;

// 1. 简化API
var regex = new RegexTool(@"\d+");
var matches = regex.Matches("abc123def");

// 2. 增强功能
var result = regex.Replace("abc123def", m => {
    return (int.Parse(m.Value) * 2).ToString();
});
```

## 多语言通用库

### 1. Hyperscan - 高性能匹配引擎

**官网**: https://github.com/intel/hyperscan

#### 特点：
- **超高性能**：处理速度可达每秒数十GB
- **多模式匹配**：同时匹配成千上万个模式
- **流式处理**：支持实时数据流匹配
- **多种语言绑定**：C、Python、Java等

#### 使用场景：
- 网络入侵检测
- 实时日志分析
- 大数据处理

### 2. RE2 - 安全正则引擎

**官网**: https://github.com/google/re2

#### 特点：
- **绝对安全**：保证线性时间匹配
- **多语言支持**：C++、Go、Python、Java等
- **生产环境验证**：Google内部广泛使用

#### 核心功能：
```python
import re2

# 安全匹配，防止回溯爆炸
pattern = re2.compile(r'(a+)+b')
result = pattern.search('aaaaaaaaac')  # 安全快速
```

## 工具类库

### 1. Regex Commons - 常用模式库

**JavaScript版本**：
```javascript
// regex-patterns库
import { patterns, validators } from 'regex-patterns';

// 使用预定义模式
const isEmail = validators.email('test@example.com');
const emailPattern = patterns.email;

// 常用模式包括：
// - email, phone, url, ip, date
// - creditCard, password, username
```

### 2. Regex Builder - 构建工具

**Python示例**：
```python
from regexbuilder import RegexBuilder

# 链式构建正则表达式
pattern = (
    RegexBuilder()
    .start_of_line()
    .digit().repeat(4)
    .literal("-")
    .digit().repeat(2)
    .literal("-")
    .digit().repeat(2)
    .end_of_line()
    .build()
)
# 生成: ^\d{4}-\d{2}-\d{2}$
```

## 选择建议

### 根据需求选择库：

1. **基础需求**：使用语言内置正则功能
2. **增强功能**：选择XRegExp（JS）、regex（Python）
3. **安全关键**：选择RE2、safe-regex
4. **高性能**：选择Hyperscan、RE2J
5. **开发体验**：选择T-Regx（PHP）、pregex（Python）

### 性能考虑：

```javascript
// 性能测试函数
function benchmark(library, pattern, testData) {
    const start = performance.now();
    for (let i = 0; i < 1000; i++) {
        library.test(pattern, testData);
    }
    return performance.now() - start;
}

// 测试不同库的性能
const results = {
    native: benchmark(RegExp, /\d+/, 'test123'),
    xregexp: benchmark(XRegExp, XRegExp('\\d+'), 'test123'),
    safe: benchmark(safeRegex, safeRegex('\\d+'), 'test123')
};
```

## 总结

正则表达式库提供了：

- **增强功能**：命名分组、递归匹配等
- **更好性能**：优化算法和实现
- **安全保证**：防止正则表达式攻击
- **开发体验**：更好的API和错误处理

> 提示：对于大多数应用，语言内置的正则功能已经足够。只有在需要特定功能或遇到性能、安全问题时，才需要考虑使用专门的正则表达式库。选择库时要考虑项目的具体需求、性能要求和维护成本。