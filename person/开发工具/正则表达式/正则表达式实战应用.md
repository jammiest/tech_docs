# 正则表达式实战应用

> 本章将深入探讨正则表达式在实际开发中的各种应用场景，提供实用的模式和技巧，帮助你将理论知识转化为实际解决问题的能力。

## 文本处理与提取

### 提取特定信息

```regex
# 提取所有电子邮件地址
/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g

# 提取URL
/https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/g

# 提取手机号码（中国）
/\b1[3-9]\d{9}\b/g

# 提取价格
/¥\s*\d+(?:\.\d{2})?|\$\s*\d+(?:\.\d{2})?|€\s*\d+(?:\.\d{2})?/gi
```

### 日志文件分析

```regex
# Apache/Nginx 日志格式
/^(\S+) (\S+) (\S+) \[([^]]+)\] "(\S+) (.*?) (\S+)" (\d+) (\d+) "([^"]*)" "([^"]*)"$/

# 提取IP地址
/\b(?:\d{1,3}\.){3}\d{1,3}\b/

# 提取HTTP状态码
/\s(\d{3})\s/

# 提取用户代理
/"([^"]*)"$/
```

## 数据验证

### 表单验证

```javascript
// 完整的表单验证函数示例
function validateForm(data) {
    const patterns = {
        username: /^[a-zA-Z0-9_-]{3,20}$/,
        email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
        password: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
        phone: /^1[3-9]\d{9}$/,
        url: /^(https?:\/\/)?([\da-z.-]+)\.([a-z.]{2,6})([\/\w .-]*)*\/?$/,
        ip: /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/
    };

    return Object.keys(patterns).every(key => 
        patterns[key].test(data[key])
    );
}
```

### 信用卡验证

```regex
# Visa: 以4开头，13或16位数字
/^4[0-9]{12}(?:[0-9]{3})?$/

# MasterCard: 以5开头，16位数字
/^5[1-5][0-9]{14}$/

# American Express: 以34或37开头，15位数字
/^3[47][0-9]{13}$/

# 通用信用卡验证（Luhn算法前）
/^(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|3[47][0-9]{13}|6(?:011|5[0-9][0-9])[0-9]{12})$/
```

## 代码处理

### HTML/XML 处理

```regex
# 提取所有标签
/<\/?[a-z][^>]*>/gi

# 提取属性值
/(\w+)=["']?((?:.(?!["']?\s+(?:\S+)=|[>"']))+.)["']?/gi

# 移除HTML标签
/<[^>]*>/g

# 提取文本内容（简单版）
/>([^<]+)</g
```

### CSS 处理

```regex
# 提取所有类选择器
/\.([a-zA-Z0-9_-]+)/g

# 提取ID选择器
/#([a-zA-Z0-9_-]+)/g

# 提取颜色值
/#([a-fA-F0-9]{6}|[a-fA-F0-9]{3})\b|rgb\((\d{1,3}),\s*(\d{1,3}),\s*(\d{1,3})\)|rgba\((\d{1,3}),\s*(\d{1,3}),\s*(\d{1,3}),\s*([01]?\.?\d*)\)/gi
```

### JavaScript 代码分析

```regex
# 提取函数定义
/function\s+([a-zA-Z_$][0-9a-zA-Z_$]*)\s*\([^)]*\)\s*\{/g

# 提取变量声明
/(?:var|let|const)\s+([a-zA-Z_$][0-9a-zA-Z_$]*)/g

# 提取字符串字面量
/('([^'\\]|\\.)*'|"([^"\\]|\\.)*")/g

# 提取注释
/\/\/.*$|\/\*[\s\S]*?\*\//gm
```

## 文本转换与格式化

### 字符串格式化

```javascript
// 驼峰命名转换
function toCamelCase(str) {
    return str.replace(/[-_\s]+(.)?/g, (_, c) => 
        c ? c.toUpperCase() : ''
    );
}

// 帕斯卡命名转换
function toPascalCase(str) {
    return str.replace(/[-_\s]+(.)?/g, (_, c) => 
        c ? c.toUpperCase() : ''
    ).replace(/^./, str[0].toUpperCase());
}

// 蛇形命名转换
function toSnakeCase(str) {
    return str.replace(/([A-Z])/g, '_$1')
              .replace(/[-_\s]+/g, '_')
              .toLowerCase()
              .replace(/^_/, '');
}
```

### 数字格式化

```javascript
// 千分位分隔
function formatNumber(num) {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
}

// 货币格式化
function formatCurrency(amount, currency = '¥') {
    return currency + formatNumber(amount.toFixed(2));
}

// 文件大小格式化
function formatFileSize(bytes) {
    const units = ['B', 'KB', 'MB', 'GB', 'TB'];
    const i = Math.floor(Math.log(bytes) / Math.log(1024));
    return (bytes / Math.pow(1024, i)).toFixed(2) + ' ' + units[i];
}
```

## 高级搜索与替换

### 复杂替换操作

```javascript
// HTML实体编码
function encodeHTML(text) {
    return text.replace(/[&<>"']/g, function(m) {
        return {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#39;'
        }[m];
    });
}

// Markdown链接转换
function markdownLinksToHTML(text) {
    return text.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2">$1</a>');
}

// 模板变量替换
function renderTemplate(template, data) {
    return template.replace(/\{\{(\w+)\}\}/g, (_, key) => 
        data[key] || ''
    );
}
```

### 批量文件处理

```bash
# 使用sed进行批量替换
sed -i 's/old_pattern/new_pattern/g' *.txt

# 使用grep查找特定内容
grep -r "pattern" /path/to/directory

# 使用awk进行复杂文本处理
awk '/pattern/ { print $1 }' file.txt
```

## 性能敏感场景优化

### 高效匹配模式

```regex
# 使用具体字符类代替通配符
# 不好: /.*abc/
# 好: /[^abc]*abc/

# 使用锚点加速匹配
/^pattern/    # 如果模式在开头
/pattern$/    # 如果模式在结尾

# 避免嵌套量词
# 危险: /(a+)+/
# 安全: /a+/
```

### 避免回溯爆炸

```regex
# 可能导致性能问题的模式
/(a|b)*c/      # 可能产生大量回溯
/(a+)+/        # 指数级回溯

# 优化方案
/[ab]*c/       # 使用字符类
/a+c/          # 简化模式
```

## 实战案例研究

### 案例1：日志分析系统

```javascript
class LogAnalyzer {
    constructor() {
        this.patterns = {
            ip: /\b(?:\d{1,3}\.){3}\d{1,3}\b/,
            timestamp: /\[\d{2}\/\w{3}\/\d{4}:\d{2}:\d{2}:\d{2} [+-]\d{4}\]/,
            method: /"(GET|POST|PUT|DELETE|HEAD|OPTIONS|PATCH)/,
            status: /\s(\d{3})\s/,
            userAgent: /"([^"]*)"$/
        };
    }

    parseLogLine(line) {
        return {
            ip: line.match(this.patterns.ip)?.[0],
            timestamp: line.match(this.patterns.timestamp)?.[0],
            method: line.match(this.patterns.method)?.[1],
            status: line.match(this.patterns.status)?.[1],
            userAgent: line.match(this.patterns.userAgent)?.[1]
        };
    }
}
```

### 案例2：数据清洗工具

```python
import re

class DataCleaner:
    def __init__(self):
        self.patterns = {
            'email': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
            'phone': r'\b1[3-9]\d{9}\b',
            'url': r'https?://(?:[-\w.]|(?:%[\da-fA-F]{2}))+',
            'html_tags': r'<[^>]+>'
        }
    
    def clean_text(self, text):
        # 移除HTML标签
        text = re.sub(self.patterns['html_tags'], '', text)
        # 标准化空白字符
        text = re.sub(r'\s+', ' ', text).strip()
        return text
    
    def extract_contacts(self, text):
        return {
            'emails': re.findall(self.patterns['email'], text),
            'phones': re.findall(self.patterns['phone'], text)
        }
```

## 调试与测试策略

### 正则表达式调试技巧

```javascript
// 调试函数
function debugRegex(pattern, text) {
    const regex = new RegExp(pattern, 'g');
    let match;
    const results = [];
    
    while ((match = regex.exec(text)) !== null) {
        results.push({
            match: match[0],
            index: match.index,
            groups: match.slice(1)
        });
    }
    
    return results;
}

// 使用示例
const text = "Contact: john@example.com, phone: 13800138000";
const emails = debugRegex(/\b[\w.%+-]+@[\w.-]+\.[a-z]{2,}\b/gi, text);
console.log(emails);
```

### 单元测试正则表达式

```javascript
// 使用Jest进行正则表达式测试
describe('Email Validation', () => {
    const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    
    test('valid emails', () => {
        const validEmails = [
            'test@example.com',
            'user.name@domain.co.uk',
            'first.last@sub.domain.com'
        ];
        
        validEmails.forEach(email => {
            expect(emailRegex.test(email)).toBe(true);
        });
    });
    
    test('invalid emails', () => {
        const invalidEmails = [
            'invalid',
            'missing@domain',
            '@domain.com',
            'user@.com'
        ];
        
        invalidEmails.forEach(email => {
            expect(emailRegex.test(email)).toBe(false);
        });
    });
});
```

## 总结

正则表达式在实际开发中的应用极其广泛，从简单的文本匹配到复杂的数据提取，都能发挥重要作用。关键要点：

- ✅ **选择合适的工具**：根据场景选择最适合的正则表达式方案
- ✅ **注重可读性**：复杂的正则表达式要添加注释或分解为多个部分
- ✅ **考虑性能**：避免回溯爆炸和低效模式
- ✅ **充分测试**：使用各种边界情况进行测试
- ✅ **保持学习**：正则表达式功能强大，总有新的技巧可以学习

> 提示：在实际项目中，建议将常用的正则表达式模式整理成工具库，方便团队共享和维护。同时，对于特别复杂的匹配需求，考虑结合其他文本处理技术来实现。