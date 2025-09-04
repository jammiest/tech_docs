# JavaScript 中的正则表达式实现

> JavaScript 提供了完整的正则表达式支持，包括创建正则表达式对象、使用正则方法以及各种匹配标志。本节将详细介绍 JavaScript 中正则表达式的使用方法和特性。

## 创建正则表达式

在 JavaScript 中有两种创建正则表达式的方式：

### 1. 字面量语法

```javascript
const regex = /pattern/flags;
```

### 2. 构造函数语法

```javascript
const regex = new RegExp('pattern', 'flags');
```

### 示例比较

```javascript
// 字面量方式
const literalRegex = /abc/gi;

// 构造函数方式
const constructorRegex = new RegExp('abc', 'gi');
```

## 正则表达式标志

JavaScript 支持以下正则表达式标志：

| 标志 | 描述 | 示例 |
|------|------|------|
| `i` | 忽略大小写 | `/abc/i` 匹配 "abc"、"ABC"、"AbC" 等 |
| `g` | 全局匹配（查找所有匹配而非在第一个匹配后停止） | `/a/g` 在 "aaa" 中找到3个匹配 |
| `m` | 多行模式（^ 和 $ 匹配每行的开头和结尾） | `/^a/m` 匹配多行文本中每行开头的 "a" |
| `s` | dotAll 模式（. 匹配包括换行符在内的任意字符） | `/a.b/s` 匹配 "a\nb" |
| `u` | Unicode 模式（启用完整的 Unicode 支持） | `/^\u{1F600}$/u` 匹配表情符号 |
| `y` | 粘性匹配（从 lastIndex 开始匹配） | `/a/y` 从指定位置开始匹配 |

## 正则表达式方法

### 1. RegExp 对象方法

| 方法 | 描述 | 示例 |
|------|------|------|
| `test()` | 测试是否匹配，返回布尔值 | `/abc/.test('abcdef')` → true |
| `exec()` | 执行搜索匹配，返回结果数组或 null | `/a(b)c/.exec('abcabc')` → ["abc", "b"] |

### 2. String 方法使用正则表达式

| 方法 | 描述 | 示例 |
|------|------|------|
| `match()` | 返回匹配结果的数组 | 'abc123'.match(/\d+/) → ["123"] |
| `matchAll()` | 返回所有匹配结果的迭代器 | [...'a1b2'.matchAll(/\d/g)] → [["1"], ["2"]] |
| `search()` | 返回第一个匹配的索引 | 'abc'.search(/b/) → 1 |
| `replace()` | 替换匹配的子串 | 'abc'.replace(/b/, 'x') → "axc" |
| `split()` | 使用正则表达式分割字符串 | 'a,b,c'.split(/,/) → ["a", "b", "c"] |

## 高级特性

### 1. 命名捕获组（ES2018）

```javascript
const regex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const result = regex.exec('2023-05-15');
console.log(result.groups.year);  // "2023"
console.log(result.groups.month); // "05"
console.log(result.groups.day);   // "15"
```

### 2. 后行断言（ES2018）

```javascript
// 正向后行断言
const price = 'Price: $100';
const amount = price.match(/(?<=\$)\d+/)[0]; // "100"

// 负向后行断言
const notPrice = 'Price: 100';
const noMatch = notPrice.match(/(?<!\$)\d+/)[0]; // "100"（匹配不在$后的数字）
```

### 3. Unicode 属性转义（ES2018）

```javascript
// 匹配任何字母字符
const unicodeRegex = /\p{L}/u;
console.log(unicodeRegex.test('a')); // true
console.log(unicodeRegex.test('π')); // true
console.log(unicodeRegex.test('字')); // true
```

### 4. dotAll 标志（ES2018）

```javascript
// 传统模式下 . 不匹配换行符
const traditional = /a.b/.test('a\nb'); // false

// s 标志下 . 匹配任何字符包括换行符
const dotAll = /a.b/s.test('a\nb'); // true
```

## 实际应用示例

### 1. 表单验证

```javascript
// 邮箱验证
function validateEmail(email) {
  const regex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  return regex.test(email);
}

// 密码强度验证（至少8字符，包含大小写字母和数字）
function validatePassword(password) {
  const regex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/;
  return regex.test(password);
}
```

### 2. 数据提取

```javascript
// 从文本中提取所有URL
function extractUrls(text) {
  const regex = /https?:\/\/[^\s/$.?#].[^\s]*/g;
  return text.match(regex) || [];
}

// 从HTML中提取所有图片src
function extractImageSrcs(html) {
  const regex = /<img[^>]+src="([^"]+)"/g;
  const srcs = [];
  let match;
  while ((match = regex.exec(html)) !== null) {
    srcs.push(match[1]);
  }
  return srcs;
}
```

### 3. 文本处理

```javascript
// 移除HTML标签
function removeHtmlTags(html) {
  return html.replace(/<[^>]+>/g, '');
}

// 驼峰转连字符
function camelToHyphen(str) {
  return str.replace(/[A-Z]/g, match => '-' + match.toLowerCase());
}

// 格式化数字（添加千位分隔符）
function formatNumber(num) {
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
}
```

## 性能优化

### 1. 预编译正则表达式

```javascript
// 不好的做法（每次调用都创建新的正则表达式）
function badCheck(str) {
  return /a|b|c/.test(str);
}

// 好的做法（预编译正则表达式）
const goodRegex = /a|b|c/;
function goodCheck(str) {
  return goodRegex.test(str);
}
```

### 2. 避免回溯爆炸

```javascript
// 危险的正则表达式（可能导致回溯爆炸）
const dangerousRegex = /(a+)+b/;

// 更安全的替代方案
const safeRegex = /a+b/;
```

### 3. 使用具体字符类

```javascript
// 不好的做法
const badRegex = /.*abc/;

// 好的做法
const goodRegex = /[^abc]*abc/;
```

## 常见问题与解决方案

### 1. 全局匹配的状态问题

```javascript
const regex = /a/g;
regex.test('abc'); // true
regex.test('abc'); // false（因为lastIndex已经前进）

// 解决方案：重置lastIndex或重新创建正则表达式
regex.lastIndex = 0;
```

### 2. 多行匹配问题

```javascript
const text = 'first\nsecond\nthird';
// 不使用m标志时，^只匹配字符串开头
text.match(/^s/g); // null

// 使用m标志后，^匹配每行开头
text.match(/^s/gm); // ["s"]（匹配"second"的s）
```

### 3. Unicode字符处理

```javascript
// 不使用u标志时，某些Unicode字符处理不正确
/^.$/.test('😊'); // false

// 使用u标志后正确处理
/^.$/u.test('😊'); // true
```

## 浏览器兼容性

JavaScript 正则表达式的高级特性在不同浏览器中的支持情况：

| 特性 | Chrome | Firefox | Safari | Edge | IE |
|------|--------|---------|--------|------|----|
| 命名捕获组 | ✅ | ✅ | ✅ | ✅ | ❌ |
| 后行断言 | ✅ | ✅ | ✅ | ✅ | ❌ |
| Unicode属性转义 | ✅ | ✅ | ✅ | ✅ | ❌ |
| dotAll标志 | ✅ | ✅ | ✅ | ✅ | ❌ |
| 粘性标志 | ✅ | ✅ | ✅ | ✅ | ❌ |

## 总结

JavaScript 中的正则表达式实现提供了强大的文本处理能力：

- **创建方式**：字面量 `/pattern/flags` 和构造函数 `new RegExp('pattern', 'flags')`
- **常用方法**：`test()`, `exec()`, `match()`, `replace()` 等
- **ES2018新特性**：命名捕获组、后行断言、Unicode属性转义、dotAll模式
- **实际应用**：表单验证、数据提取、文本处理等
- **性能优化**：预编译正则、避免回溯爆炸、使用具体字符类

> 提示：对于复杂的文本处理需求，可以考虑结合多个简单的正则表达式分步骤处理，或者使用专门的解析器库。现代JavaScript的正则表达式功能已经非常强大，合理使用可以大大提高开发效率。