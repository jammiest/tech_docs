# Python 函数速查手册

## 内置函数
```python
# 类型转换
str(obj)                    # 转换为字符串
int(obj)                    # 转换为整数
float(obj)                  # 转换为浮点数
bool(obj)                   # 转换为布尔值
list(iterable)              # 转换为列表
tuple(iterable)             # 转换为元组
dict(iterable)              # 转换为字典
set(iterable)               # 转换为集合

# 数学运算
abs(x)                      # 绝对值
round(x, n)                 # 四舍五入
pow(x, y)                   # 幂运算
divmod(a, b)                # 返回商和余数
max(iterable)               # 最大值
min(iterable)               # 最小值
sum(iterable)               # 求和

# 输入输出
print(*objects, sep=' ')    # 打印输出
input(prompt)               # 获取用户输入
open(file, mode='r')        # 打开文件
len(obj)                    # 获取长度
range(start, stop, step)    # 生成序列

# 迭代相关
enumerate(iterable)         # 添加索引的枚举
zip(*iterables)             # 并行迭代
sorted(iterable, key=None, reverse=False)  # 排序
reversed(sequence)          # 反转序列

# 类型检查
type(obj)                   # 返回类型
isinstance(obj, classinfo)  # 类型检查
issubclass(class, classinfo) # 子类检查
callable(obj)               # 是否可调用

# 其他常用
id(obj)                     # 获取对象id
hash(obj)                   # 获取哈希值
help(obj)                   # 查看帮助
dir(obj)                    # 查看属性列表
```

## 字符串方法
```python
s.strip()                   # 去除首尾空白字符
s.split(sep=None)           # 分割字符串
s.join(iterable)            # 连接字符串
s.replace(old, new)         # 替换子串
s.find(sub)                 # 查找子串位置
s.index(sub)                # 查找子串位置(找不到报错)
s.startswith(prefix)        # 是否以指定字符串开头
s.endswith(suffix)          # 是否以指定字符串结尾
s.lower()                   # 转换为小写
s.upper()                   # 转换为大写
s.title()                   # 单词首字母大写
s.isdigit()                 # 是否全为数字
s.isalpha()                 # 是否全为字母
s.isalnum()                 # 是否全为字母或数字
s.format(*args, **kwargs)   # 字符串格式化
f"{variable}"               # f-string格式化
```

## 列表方法
```python
# 增删改查
lst.append(x)               # 末尾添加元素
lst.extend(iterable)        # 扩展列表
lst.insert(i, x)            # 指定位置插入
lst.remove(x)               # 删除第一个匹配元素
lst.pop([i])                # 删除并返回指定位置元素
lst.clear()                 # 清空列表
lst.index(x)                # 返回元素索引
lst.count(x)                # 统计元素出现次数

# 排序操作
lst.sort(key=None, reverse=False)  # 原地排序
lst.reverse()               # 原地反转
sorted_list = sorted(lst)   # 返回新排序列表

# 列表推导式
[i*2 for i in range(10)]                    # 简单推导式
[i for i in range(10) if i % 2 == 0]        # 带条件推导式
[i if i % 2 == 0 else -i for i in range(10)] # 条件表达式
```

## 字典方法
```python
# 基本操作
dct.keys()                  # 返回所有键
dct.values()                # 返回所有值
dct.items()                 # 返回键值对
dct.get(key, default)       # 安全获取值
dct.setdefault(key, default) # 键不存在时设置默认值
dct.update(other)           # 更新字典
dct.pop(key, default)       # 删除并返回值
dct.popitem()               # 删除并返回最后键值对
dct.clear()                 # 清空字典

# 字典推导式
{key: value for key, value in iterable}
{x: x**2 for x in range(5)}
```

## 集合方法
```python
s.add(x)                    # 添加元素
s.remove(x)                 # 删除元素(不存在报错)
s.discard(x)                # 删除元素(不存在不报错)
s.pop()                     # 随机删除并返回元素
s.clear()                   # 清空集合
s.union(other)              # 并集
s.intersection(other)       # 交集
s.difference(other)         # 差集
s.symmetric_difference(other) # 对称差集
s.issubset(other)           # 子集判断
s.issuperset(other)         # 超集判断
```

## 文件操作
```python
# 打开模式: r(读), w(写), a(追加), b(二进制), +(读写)
with open('file.txt', 'r') as f:
    content = f.read()              # 读取全部内容
    line = f.readline()             # 读取一行
    lines = f.readlines()           # 读取所有行

with open('file.txt', 'w') as f:
    f.write('content')              # 写入内容
    f.writelines(['line1', 'line2']) # 写入多行
```

## 日期时间
```python
from datetime import datetime, date, time, timedelta

now = datetime.now()                # 当前时间
dt = datetime(2023, 12, 25, 10, 30) # 创建指定时间
dt.strftime('%Y-%m-%d %H:%M:%S')    # 格式化时间
datetime.strptime('2023-12-25', '%Y-%m-%d') # 解析时间

delta = timedelta(days=7)           # 时间间隔
future = now + delta                # 时间计算
```

## 错误处理
```python
try:
    # 可能出错的代码
    result = 10 / 0
except ZeroDivisionError as e:
    print("除零错误:", e)
except Exception as e:
    print("其他错误:", e)
else:
    print("没有错误时执行")
finally:
    print("无论是否出错都执行")

# 抛出异常
raise ValueError("错误信息")
```

## 函数定义
```python
# 基本函数
def function_name(arg1, arg2=default):
    """函数文档字符串"""
    return result

# 可变参数
def func(*args, **kwargs):
    print(args)    # 元组
    print(kwargs)  # 字典

# 匿名函数
lambda x: x * 2

# 类型提示(Python 3.5+)
def greet(name: str) -> str:
    return f"Hello {name}"
```

## 模块和包
```python
import module_name
from module_name import function_name
from module_name import *
import module_name as alias

# 包结构
# package/
#     __init__.py
#     module1.py
#     module2.py

from package import module1
from package.module1 import function_name
```

## 面向对象
```python
class MyClass:
    # 类属性
    class_attr = "value"
    
    def __init__(self, arg1, arg2):
        # 实例属性
        self.arg1 = arg1
        self.arg2 = arg2
    
    def instance_method(self):
        return self.arg1
    
    @classmethod
    def class_method(cls):
        return cls.class_attr
    
    @staticmethod
    def static_method():
        return "static"
    
    # 特殊方法
    def __str__(self):
        return f"MyClass({self.arg1})"

# 继承
class ChildClass(MyClass):
    def __init__(self, arg1, arg2, arg3):
        super().__init__(arg1, arg2)
        self.arg3 = arg3
```

## 迭代器和生成器
```python
# 迭代器
class MyIterator:
    def __iter__(self):
        return self
    def __next__(self):
        # 返回下一个值或抛出StopIteration
        pass

# 生成器函数
def generator_function():
    yield 1
    yield 2
    yield 3

# 生成器表达式
gen = (x**2 for x in range(10))
```

## 装饰器
```python
def decorator(func):
    def wrapper(*args, **kwargs):
        # 前置处理
        result = func(*args, **kwargs)
        # 后置处理
        return result
    return wrapper

@decorator
def my_function():
    pass

# 带参数的装饰器
def param_decorator(param):
    def actual_decorator(func):
        def wrapper(*args, **kwargs):
            print(f"Parameter: {param}")
            return func(*args, **kwargs)
        return wrapper
    return actual_decorator
```

## 常用内置模块
```python
# os - 操作系统接口
import os
os.getcwd()                 # 当前工作目录
os.listdir()                # 列出目录内容
os.path.join('dir', 'file') # 路径拼接
os.path.exists('path')      # 路径是否存在

# sys - 系统相关参数和函数
import sys
sys.argv                    # 命令行参数
sys.exit()                  # 退出程序

# re - 正则表达式
import re
re.findall(pattern, string) # 查找所有匹配
re.search(pattern, string)  # 搜索匹配
re.sub(pattern, repl, string) # 替换

# json - JSON处理
import json
json.dumps(obj)             # 对象转JSON字符串
json.loads(string)          # JSON字符串转对象

# math - 数学运算
import math
math.sqrt(x)                # 平方根
math.sin(x)                 # 正弦
math.pi                     # 圆周率

# random - 随机数
import random
random.random()             # 0-1随机数
random.randint(a, b)        # 整数随机数
random.choice(sequence)     # 随机选择
```