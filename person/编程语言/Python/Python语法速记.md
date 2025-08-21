# Python 基础速记手册

## 变量定义与访问

```python
# 基本定义
a = 10
b = "hello"
c = True

# 多变量赋值
x, y = 1, 2
x, y = y, x  # 交换变量

# 常量(约定全大写)
PI = 3.14
MAX_CONNECTIONS = 100

# 特殊变量
_ = 42  # 忽略值
__private = "private"  # 私有变量(约定)
```

## 基本数据类型

```python
# 数字类型
int_var = 10
float_var = 3.14
complex_var = 1 + 2j

# 布尔类型
bool_var = True  # 或 False

# 序列类型
str_var = "hello"
bytes_var = b"hello"
list_var = [1, 2, 3]
tuple_var = (1, 2, 3)

# 映射类型
dict_var = {"a": 1, "b": 2}

# 集合类型
set_var = {1, 2, 3}
frozenset_var = frozenset({1, 2, 3})
```

## 复合类型操作

```python
# 字符串操作
s = "Python"
s[0]  # 'P'
s[1:4]  # 'yth'
len(s)  # 6
"Py" in s  # True
s.lower()  # 'python'
s.replace("P", "J")  # 'Jython'

# 列表操作
lst = [1, 2, 3]
lst.append(4)  # [1, 2, 3, 4]
lst.extend([5, 6])  # [1, 2, 3, 4, 5, 6]
lst.pop()  # 6 (返回并删除最后一个元素)
lst[1:3] = [7, 8]  # [1, 7, 8, 4, 5]

# 字典操作
d = {"a": 1, "b": 2}
d["c"] = 3  # 添加/修改
d.get("d", 0)  # 0 (获取值，不存在返回默认值)
d.keys()  # dict_keys(['a', 'b', 'c'])
d.values()  # dict_values([1, 2, 3])

# 集合操作
s1 = {1, 2, 3}
s2 = {3, 4, 5}
s1 | s2  # 并集 {1, 2, 3, 4, 5}
s1 & s2  # 交集 {3}
s1 - s2  # 差集 {1, 2}
```

## 类与对象

```python
# 类定义
class Person:
    # 类属性
    species = "Homo sapiens"
    
    # 初始化方法
    def __init__(self, name, age):
        self.name = name  # 实例属性
        self.age = age
    
    # 实例方法
    def greet(self):
        return f"Hello, I'm {self.name}"
    
    # 类方法
    @classmethod
    def from_birth_year(cls, name, birth_year):
        age = datetime.date.today().year - birth_year
        return cls(name, age)
    
    # 静态方法
    @staticmethod
    def is_adult(age):
        return age >= 18

# 继承
class Student(Person):
    def __init__(self, name, age, student_id):
        super().__init__(name, age)
        self.student_id = student_id
    
    # 方法重写
    def greet(self):
        return f"Hi, I'm student {self.name}"

# 使用类
p = Person("Alice", 25)
s = Student("Bob", 20, "S12345")
```

## 函数

```python
# 基本函数
def add(a, b):
    return a + b

# 默认参数
def greet(name="World"):
    return f"Hello, {name}!"

# 可变位置参数
def sum_all(*args):
    return sum(args)

# 可变关键字参数
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

# 混合参数
def complex_func(a, b=2, *args, **kwargs):
    pass

# lambda函数
square = lambda x: x ** 2

# 闭包
def make_multiplier(n):
    def multiplier(x):
        return x * n
    return multiplier

# 装饰器
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")
```

## 控制结构

```python
# if-elif-else
x = 10
if x > 10:
    print("Greater")
elif x == 10:
    print("Equal")
else:
    print("Less")

# for循环
for i in range(5):  # 0到4
    print(i)

for item in [1, 2, 3]:
    print(item)

for key, value in {"a": 1, "b": 2}.items():
    print(key, value)

# while循环
count = 0
while count < 5:
    print(count)
    count += 1

# 循环控制
for i in range(10):
    if i == 3:
        continue  # 跳过本次循环
    if i == 7:
        break  # 终止循环
    print(i)
else:  # 循环正常结束时执行
    print("Loop completed")

# 异常处理
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
except Exception as e:
    print(f"Error: {e}")
else:
    print("No errors")
finally:
    print("Always executed")
```

## 常用内置函数

```python
# 类型转换
int("10"), float("3.14"), str(10)
list("hello"), tuple([1, 2, 3]), set([1, 2, 2])
dict([("a", 1), ("b", 2)])

# 数学运算
abs(-5), round(3.14159, 2), pow(2, 3)
min([1, 2, 3]), max([1, 2, 3]), sum([1, 2, 3])

# 序列操作
len("hello"), sorted([3, 1, 2]), reversed([1, 2, 3])
range(5), enumerate(["a", "b"]), zip([1, 2], ["a", "b"])

# 输入输出
input("Enter something: ")
print("Hello", "World", sep=", ", end="!\n")

# 文件操作
open("file.txt", "r")  # 模式: r(读), w(写), a(追加), b(二进制)

# 对象操作
isinstance(10, int), issubclass(Student, Person)
hasattr(p, "name"), getattr(p, "name"), setattr(p, "age", 30)
callable(add), dir(p), vars(p)

# 其他
help(str), type(10), id(10)
globals(), locals()
```

## 常用模块函数

```python
# os模块
os.getcwd(), os.listdir(), os.path.join("dir", "file.txt")
os.path.exists("file.txt"), os.path.isfile("file.txt")

# sys模块
sys.argv, sys.exit(), sys.path

# math模块
math.sqrt(16), math.sin(math.pi/2), math.ceil(3.2)

# random模块
random.random(), random.randint(1, 10), random.choice([1, 2, 3])

# datetime模块
datetime.datetime.now(), datetime.timedelta(days=1)
date.strftime("%Y-%m-%d"), datetime.strptime("2023-01-01", "%Y-%m-%d")

# json模块
json.dumps({"a": 1}), json.loads('{"a": 1}')

# re模块
re.search(r"\d+", "abc123"), re.findall(r"\d+", "a1b22c333")
re.sub(r"\d+", "X", "a1b22c333")

# collections模块
collections.defaultdict(int), collections.Counter("hello")
collections.deque([1, 2, 3]), collections.namedtuple("Point", ["x", "y"])
```

## 特殊方法

```python
# 比较方法
__eq__, __ne__, __lt__, __gt__, __le__, __ge__

# 数学运算
__add__, __sub__, __mul__, __truediv__, __floordiv__, __mod__

# 其他运算
__and__, __or__, __xor__, __invert__, __lshift__, __rshift__

# 容器方法
__len__, __getitem__, __setitem__, __delitem__, __contains__

# 调用和属性
__call__, __getattr__, __setattr__, __delattr__, __getattribute__

# 字符串表示
__str__, __repr__, __format__

# 迭代器协议
__iter__, __next__

# 上下文管理
__enter__, __exit__
```