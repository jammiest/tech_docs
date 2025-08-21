# C++ 基础速记手册

## 变量定义与访问

```cpp
// 基本定义
int a = 10;
double b = 3.14;
char c = 'A';
bool d = true;
auto e = "Hello";  // 自动类型推导

// 常量定义
const int MAX_SIZE = 100;
constexpr double PI = 3.1415926;  // 编译期常量

// 指针与引用
int* ptr = &a;     // 指针
int& ref = a;      // 引用
const int* cptr = &a;  // 指向常量的指针
int* const cp = &a;    // 常量指针

// 类型转换
static_cast<double>(a)  // 静态转换
dynamic_cast<Derived*>(basePtr)  // 动态转换
const_cast<int&>(constInt)  // 常量转换
reinterpret_cast<void*>(ptr)  // 重新解释
```

## 数据类型

```cpp
// 基本类型
bool, char, short, int, long, long long
float, double, long double
void, nullptr_t

// 类型修饰符
signed, unsigned, const, volatile, mutable

// 类型别名
typedef int MyInt;
using MyDouble = double;  // C++11

// 类型信息
typeid(a).name()
sizeof(int)
alignof(double)
```

## 数组与字符串

```cpp
// 数组定义
int arr1[5] = {1, 2, 3};
int arr2[] = {4, 5, 6};
std::array<int, 3> stdArr = {7, 8, 9};  // C++11

// 字符串
char str1[] = "Hello";
std::string str2 = "World";
const char* str3 = "C++";

// 常用操作
std::strlen(str1)  // C风格字符串长度
str2.size(), str2.length()  // string长度
str2.append("!"), str2 += "!"  // 追加
str2.substr(1, 3)  // 子串
str2.find("or")  // 查找
std::to_string(123)  // 数字转字符串
std::stoi("456")  // 字符串转数字
```

## 类与对象

```cpp
// 类定义
class Person {
private:
    std::string name;
    int age;
    
public:
    // 构造函数
    Person() : name(""), age(0) {}  // 默认构造
    Person(std::string n, int a) : name(n), age(a) {}
    
    // 拷贝构造
    Person(const Person& other) : name(other.name), age(other.age) {}
    
    // 移动构造 (C++11)
    Person(Person&& other) noexcept : name(std::move(other.name)), age(other.age) {}
    
    // 析构函数
    ~Person() {}
    
    // 成员函数
    void setName(std::string n) { name = n; }
    std::string getName() const { return name; }
    
    // 静态成员
    static int count;
    
    // 友元函数
    friend void printPerson(const Person& p);
};

// 静态成员初始化
int Person::count = 0;

// 继承
class Student : public Person {
private:
    std::string studentId;
    
public:
    using Person::Person;  // 继承构造函数
    
    // 重写方法
    void printInfo() const {
        std::cout << "Student: " << getName() << std::endl;
    }
};

// 多态
class Shape {
public:
    virtual double area() const = 0;  // 纯虚函数
    virtual ~Shape() = default;      // 虚析构
};

class Circle : public Shape {
    double radius;
public:
    double area() const override { return 3.14 * radius * radius; }
};

// 接口 (抽象类)
class IPrintable {
public:
    virtual void print() const = 0;
};

// 对象操作
Person p1("Alice", 25);
Person p2 = p1;  // 拷贝构造
auto p3 = std::move(p1);  // 移动构造 (C++11)
```

## 函数

```cpp
// 函数定义
int add(int a, int b) {
    return a + b;
}

// 默认参数
void log(std::string msg, int level = 1) {
    std::cout << "[" << level << "] " << msg << std::endl;
}

// 函数重载
void print(int i) { std::cout << i; }
void print(double d) { std::cout << d; }
void print(std::string s) { std::cout << s; }

// 内联函数
inline int max(int a, int b) {
    return a > b ? a : b;
}

// Lambda表达式 (C++11)
auto square = int x { return x * x; };
auto sum = auto a, auto b { return a + b; };  // C++14

// 函数指针
int (*funcPtr)(int, int) = add;

// 模板函数
template <typename T>
T max(T a, T b) {
    return a > b ? a : b;
}

// 可变参数模板 (C++11)
template<typename... Args>
void printAll(Args... args) {
    (std::cout << ... << args) << std::endl;  // C++17折叠表达式
}
```

## 控制结构

```cpp
// 条件语句
if (a > b) {
    // ...
} else if (a == b) {
    // ...
} else {
    // ...
}

// switch语句
switch (value) {
    case 1:
        // ...
        break;
    case 2:
        // ...
        break;
    default:
        // ...
}

// 循环语句
for (int i = 0; i < 10; ++i) {
    // ...
}

for (auto& item : container) {  // 范围for (C++11)
    // ...
}

while (condition) {
    // ...
}

do {
    // ...
} while (condition);

// 跳转语句
break;
continue;
return;
goto label;  // 慎用
```

## 标准库常用功能

### 容器

```cpp
// 序列容器
std::vector<int> vec = {1, 2, 3};
std::list<int> lst = {4, 5, 6};
std::deque<int> deq = {7, 8, 9};
std::array<int, 3> arr = {10, 11, 12};  // C++11

// 关联容器
std::set<int> s = {1, 2, 3};
std::map<std::string, int> m = {{"a", 1}, {"b", 2}};
std::unordered_set<int> us = {4, 5, 6};  // C++11
std::unordered_map<std::string, int> um = {{"c", 3}, {"d", 4}};  // C++11

// 容器适配器
std::stack<int> stk;
std::queue<int> que;
std::priority_queue<int> pq;

// 常用操作
vec.push_back(4);  // 添加元素
vec.size();        // 元素数量
vec.empty();       // 是否为空
vec.front();       // 首元素
vec.back();        // 尾元素
vec.begin();       // 迭代器开始
vec.end();         // 迭代器结束
```

### 算法

```cpp
#include <algorithm>

// 排序与查找
std::sort(vec.begin(), vec.end());
std::binary_search(vec.begin(), vec.end(), 5);
auto it = std::find(vec.begin(), vec.end(), 3);

// 数值算法
int sum = std::accumulate(vec.begin(), vec.end(), 0);
int product = std::accumulate(vec.begin(), vec.end(), 1, std::multiplies<int>());

// 变换
std::transform(vec.begin(), vec.end(), vec.begin(), int x { return x * 2; });

// 其他常用算法
std::reverse(vec.begin(), vec.end());
std::count(vec.begin(), vec.end(), 2);
std::max_element(vec.begin(), vec.end());
std::min_element(vec.begin(), vec.end());
std::fill(vec.begin(), vec.end(), 0);
```

### 智能指针 (C++11)

```cpp
#include <memory>

std::unique_ptr<int> uptr(new int(10));  // 独占所有权
auto uptr2 = std::make_unique<int>(20);  // C++14

std::shared_ptr<int> sptr(new int(30));  // 共享所有权
auto sptr2 = std::make_shared<int>(40);  // 更高效

std::weak_ptr<int> wptr = sptr;  // 弱引用，不增加计数
```

### 多线程 (C++11)

```cpp
#include <thread>
#include <mutex>
#include <future>

// 线程创建
std::thread t({
    std::cout << "Hello from thread" << std::endl;
});
t.join();

// 互斥锁
std::mutex mtx;
mtx.lock();
// 临界区代码
mtx.unlock();

// 更安全的锁管理
std::lock_guard<std::mutex> lock(mtx);  // 离开作用域自动释放
std::unique_lock<std::mutex> ulock(mtx);  // 更灵活的锁

// 异步操作
auto future = std::async(std::launch::async, {
    return 42;
});
int result = future.get();
```

### 文件操作

```cpp
#include <fstream>

// 文本文件读写
std::ifstream in("input.txt");
std::string line;
while (std::getline(in, line)) {
    // 处理每一行
}
in.close();

std::ofstream out("output.txt");
out << "Hello, file!" << std::endl;
out.close();

// 二进制文件
std::fstream bin("data.bin", std::ios::binary | std::ios::in | std::ios::out);
int num = 123;
bin.write(reinterpret_cast<char*>(&num), sizeof(num));
bin.read(reinterpret_cast<char*>(&num), sizeof(num));
bin.close();
```

## 异常处理

```cpp
try {
    // 可能抛出异常的代码
    if (error) {
        throw std::runtime_error("Something went wrong");
    }
} catch (const std::runtime_error& e) {
    std::cerr << "Runtime error: " << e.what() << std::endl;
} catch (const std::exception& e) {
    std::cerr << "Standard exception: " << e.what() << std::endl;
} catch (...) {
    std::cerr << "Unknown exception" << std::endl;
}

// 自定义异常
class MyException : public std::exception {
public:
    const char* what() const noexcept override {
        return "My custom exception";
    }
};
```

## 其他重要特性

### 移动语义 (C++11)

```cpp
// 右值引用
int&& rref = 42;

// 移动构造
class Resource {
    int* data;
public:
    Resource(Resource&& other) noexcept : data(other.data) {
        other.data = nullptr;  // 转移所有权
    }
};

// 完美转发
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));  // 保持值类别
}
```

### 类型推导 (C++11/14)

```cpp
auto x = 42;  // int
auto y = 3.14;  // double

decltype(x) z = x;  // 推导类型

// C++14
auto lambda = auto a, auto b { return a + b; };
```

### 结构化绑定 (C++17)

```cpp
std::pair<int, std::string> p{1, "one"};
auto [num, str] = p;  // num=1, str="one"

std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
for (const auto& [key, value] : m) {
    std::cout << key << ": " << value << std::endl;
}
```

### 概念 (C++20)

```cpp
template<typename T>
concept Integral = std::is_integral_v<T>;

template<Integral T>
T add(T a, T b) {
    return a + b;
}
```