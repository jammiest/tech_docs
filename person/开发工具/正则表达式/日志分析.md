# 正则表达式日志分析指南

> 日志分析是正则表达式的重要应用场景，能够从海量日志数据中提取有价值的信息。本节将详细介绍如何使用正则表达式进行高效的日志分析。

## 常见日志格式解析

### 1. Apache/Nginx 访问日志

```regex
# 通用格式解析
^(\S+) (\S+) (\S+) \[([^\]]+)\] "(\S+) (.*?) (\S+)" (\d+) (\d+) "([^"]*)" "([^"]*)"

# 分组说明
1: 客户端IP
2: 标识符（通常为-）
3: 用户标识（通常为-）
4: 时间戳
5: HTTP方法
6: 请求URL
7: HTTP协议
8: 状态码
9: 响应大小
10: Referer
11: 用户代理
```

### 2. 错误日志格式

```regex
# Apache错误日志
^\[([^\]]+)\] \[([^\]]+)\] \[client (\S+)\] (.*)$

# Nginx错误日志
^(\d{4}/\d{2}/\d{2} \d{2}:\d{2}:\d{2}) \[(\w+)\] (\d+)#\d+: \*(?\d+) (\S+): (.*)$
```

### 3. 自定义应用日志

```regex
# 常见格式
^\[(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\] \[(\w+)\] (\S+): (.*)$

# 分组说明
1: 时间戳
2: 日志级别
3: 类名/模块名
4: 日志消息
```

## 关键信息提取

### 1. 提取访问统计

```regex
# 提取访问最多的URL
"GET (\S+) HTTP"

# 提取状态码分布
"HTTP/\d\.\d" (\d{3})

# 提取客户端IP
^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}

# 提取用户代理
"([^"]*)"$
```

### 2. 错误分析

```regex
# 提取5xx错误
"HTTP/\d\.\d" (5\d{2})

# 提取404错误
"GET (\S+) HTTP" 404

# 提取错误详情
\[error\] (.*)
```

### 3. 性能分析

```regex
# 提取响应时间（如果有记录）
(\d+)ms$

# 提取大文件请求
"GET (.*?\.(?:mp4|zip|rar|exe)) HTTP" 200 (\d{5,})

# 提取慢请求
(\d+\.\d+) seconds
```

## 实际分析案例

### 1. 分析访问频率

```python
import re
from collections import defaultdict

def analyze_access(log_file):
    ip_counts = defaultdict(int)
    url_counts = defaultdict(int)
    
    pattern = r'^(\S+).*"GET (\S+) HTTP'
    
    with open(log_file) as f:
        for line in f:
            match = re.match(pattern, line)
            if match:
                ip, url = match.groups()
                ip_counts[ip] += 1
                url_counts[url] += 1
    
    return ip_counts, url_counts
```

### 2. 检测异常请求

```javascript
function detectAnomalies(logs) {
    const anomalies = [];
    const pattern = /"GET (\S+) HTTP" (4\d{2})/g;
    
    let match;
    while ((match = pattern.exec(logs)) !== null) {
        const [_, url, status] = match;
        if (status === '404') {
            anomalies.push({
                type: 'Not Found',
                url,
                line: match.index
            });
        } else if (status === '403') {
            anomalies.push({
                type: 'Forbidden',
                url,
                line: match.index
            });
        }
    }
    
    return anomalies;
}
```

### 3. 提取会话信息

```java
import java.util.regex.*;
import java.util.*;

public class SessionExtractor {
    public static Map<String, List<String>> extractSessions(String logs) {
        Map<String, List<String>> sessions = new HashMap<>();
        Pattern pattern = Pattern.compile("\\b([0-9a-f]{32})\\b.*?\"GET (\\S+) HTTP");
        
        Matcher matcher = pattern.matcher(logs);
        while (matcher.find()) {
            String sessionId = matcher.group(1);
            String url = matcher.group(2);
            
            sessions.computeIfAbsent(sessionId, k -> new ArrayList<>()).add(url);
        }
        
        return sessions;
    }
}
```

## 高级分析技巧

### 1. 多行日志处理

```regex
# 匹配多行异常堆栈
^\[ERROR\].*?(\n\tat .*)*

# 匹配事务日志
^BEGIN TRANSACTION.*?(\n.*?)*?END TRANSACTION
```

### 2. 时间范围过滤

```python
import re
from datetime import datetime

def filter_logs_by_time(logs, start_time, end_time):
    pattern = r'\[(\d{2}/\w{3}/\d{4}:\d{2}:\d{2}:\d{2})'
    filtered = []
    
    for line in logs.split('\n'):
        match = re.search(pattern, line)
        if match:
            log_time = datetime.strptime(match.group(1), '%d/%b/%Y:%H:%M:%S')
            if start_time <= log_time <= end_time:
                filtered.append(line)
    
    return filtered
```

### 3. 性能瓶颈检测

```regex
# 检测慢查询
execution_time=(\d+\.\d+)s

# 检测高内存使用
memory_usage=(\d+)MB

# 检测高CPU使用
cpu_load=(\d+\.\d+)
```

## 日志可视化准备

### 1. 提取时序数据

```python
import re

def extract_time_series(logs, pattern):
    data = []
    for line in logs.split('\n'):
        match = re.search(pattern, line)
        if match:
            timestamp = match.group(1)
            value = float(match.group(2))
            data.append((timestamp, value))
    return data
```

### 2. 生成统计报表

```javascript
function generateStats(logs) {
    const stats = {
        total: 0,
        byStatus: {},
        byHour: Array(24).fill(0),
        topUrls: {}
    };
    
    // 匹配日志行
    const linePattern = /^.*?"(\S+) (\S+) (\S+)" (\d+)/g;
    let match;
    
    while ((match = linePattern.exec(logs)) !== null) {
        const [_, method, url, protocol, status] = match;
        stats.total++;
        
        // 按状态码统计
        stats.byStatus[status] = (stats.byStatus[status] || 0) + 1;
        
        // 按小时统计
        const hourMatch = /\[.*?(\d{2}):\d{2}:\d{2}/.exec(match[0]);
        if (hourMatch) {
            const hour = parseInt(hourMatch[1]);
            stats.byHour[hour]++;
        }
        
        // 统计URL
        stats.topUrls[url] = (stats.topUrls[url] || 0) + 1;
    }
    
    return stats;
}
```

## 实用工具函数

### 1. 日志过滤器

```python
def filter_logs(logs, conditions):
    """
    conditions = {
        'status': '404',
        'method': 'GET',
        'ip': '192.168.1.*'
    }
    """
    filtered = []
    pattern = re.compile(
        r'^(?P<ip>\S+).*?"(?P<method>\S+) (?P<url>\S+) (?P<protocol>\S+)" (?P<status>\d+)'
    )
    
    for line in logs:
        match = pattern.match(line)
        if match:
            matched = True
            for key, value in conditions.items():
                if key in match.groupdict():
                    if isinstance(value, str) and '*' in value:
                        # 处理通配符
                        value_pattern = value.replace('.', '\\.').replace('*', '.*')
                        if not re.match(value_pattern, match.group(key)):
                            matched = False
                            break
                    elif match.group(key) != value:
                        matched = False
                        break
            if matched:
                filtered.append(line)
    
    return filtered
```

### 2. 实时日志监控

```python
import re
import tailer

def monitor_log(log_file, alert_patterns):
    """
    alert_patterns = [
        {'pattern': '500', 'message': 'Server error'},
        {'pattern': 'SQL error', 'message': 'Database error'}
    ]
    """
    for line in tailer.follow(open(log_file)):
        for alert in alert_patterns:
            if re.search(alert['pattern'], line):
                print(f"ALERT: {alert['message']} - {line.strip()}")
```

## 性能优化建议

1. **预编译正则表达式**：对于重复使用的模式
   ```python
   pattern = re.compile(r'...')
   pattern.match(line)
   ```

2. **使用非捕获分组**：当不需要捕获内容时
   ```regex
   (?:pattern) 替代 (pattern)
   ```

3. **避免回溯爆炸**：简化复杂的正则表达式
   ```regex
   .*? 替代 .*
   ```

4. **分步骤处理**：复杂分析可分多步进行

## 常见问题解决方案

### 1. 处理多格式日志

```python
def parse_multiformat_logs(logs):
    patterns = [
        r'^\[(?P<timestamp>.*?)\] \[(?P<level>\w+)\] (?P<message>.*)$',
        r'^(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (?P<level>\w+) (?P<message>.*)$',
        r'^(?P<ip>\S+) - - \[(?P<timestamp>.*?)\] "(?P<method>\S+) (?P<url>\S+) (?P<protocol>\S+)" (?P<status>\d+)'
    ]
    
    for line in logs:
        for pattern in patterns:
            match = re.match(pattern, line)
            if match:
                yield match.groupdict()
                break
        else:
            yield {'raw': line}
```

### 2. 处理非标准时间格式

```python
def normalize_timestamp(timestamp):
    formats = [
        '%d/%b/%Y:%H:%M:%S',  # Apache格式
        '%Y-%m-%d %H:%M:%S',  # ISO格式
        '%b %d %H:%M:%S',     # syslog格式
        '%m/%d/%Y %I:%M:%S %p' # 美国格式
    ]
    
    for fmt in formats:
        try:
            return datetime.strptime(timestamp, fmt)
        except ValueError:
            continue
    return timestamp  # 无法解析则返回原始值
```

### 3. 处理大日志文件

```python
def process_large_file(filename, chunk_size=1024*1024):
    with open(filename) as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            
            # 处理最后一个完整行
            while True:
                next_char = f.read(1)
                if not next_char or next_char == '\n':
                    break
                chunk += next_char
            
            for line in chunk.split('\n'):
                # 处理每一行
                process_line(line)
```

## 总结

正则表达式在日志分析中的应用要点：

- **格式解析**：理解日志格式并设计匹配模式
- **关键提取**：从日志中提取有价值的信息
- **异常检测**：识别错误和异常模式
- **统计分析**：生成访问统计和性能指标
- **可视化准备**：格式化数据用于可视化展示

> 提示：对于生产环境的日志分析，建议结合专门的日志分析工具（如ELK Stack、Splunk等）和正则表达式，以获得最佳的分析效果和性能。