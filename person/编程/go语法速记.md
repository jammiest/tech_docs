# Go 语言基础速记

## 变量定义与访问

```go
// 基本定义
var a int = 10
var b = "hello"
c := true // 短声明

// 多变量
var x, y int = 1, 2
var (  // 分组声明
    name string = "Go"
    version float64 = 1.18
)

// 常量
const PI = 3.14
const (
    StatusOK = 200
    StatusNotFound = 404
)

// 特殊变量
_ = 42 // 忽略值
```

## 基本类型

```go
bool, string, 
int, int8, int16, int32, int64, 
uint, uint8, uint16, uint32, uint64, uintptr, 
float32, float64, 
complex64, complex128, 
byte // uint8 别名, rune // int32 别名
```

## 复合类型

```go
// 数组
var arr [3]int = [3]int{1, 2, 3}
arr2 := [...]int{4, 5, 6} // 编译器推导长度

// 切片
slice := []int{1, 2, 3}
slice = append(slice, 4) // 追加
sub := slice[1:3] // 切片操作

// 映射
m := map[string]int{"a": 1, "b": 2}
m["c"] = 3 // 添加/修改
delete(m, "a") // 删除

// 结构体
type Person struct {
    Name string
    Age  int
}
p := Person{Name: "Alice", Age: 30}
```

## 类与接口

```go
// 方法定义
func (p *Person) SayHello() {
    fmt.Printf("Hello, I'm %s\n", p.Name)
}

// 接口
type Speaker interface {
    Speak() string
}

// 实现接口
func (p Person) Speak() string {
    return "Hi!"
}

// 类型断言
var s Speaker = Person{}
if person, ok := s.(Person); ok {
    person.SayHello()
}
```

## 函数

```go
// 基本函数
func add(a, b int) int {
    return a + b
}

// 多返回值
func swap(a, b int) (int, int) {
    return b, a
}

// 命名返回值
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return // 裸返回
}

// 可变参数
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}

// 匿名函数
func() {
    fmt.Println("Anonymous function")
}()

// 闭包
func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}
```

## 控制结构

```go
// if
if x := 10; x > 5 {
    fmt.Println("x is greater than 5")
}

// for
for i := 0; i < 10; i++ { /*...*/ }
for i, v := range slice { /*...*/ } // range循环

// switch
switch os := runtime.GOOS; os {
case "darwin":
    fmt.Println("OS X")
default:
    fmt.Printf("%s\n", os)
}

// defer
func readFile() {
    file, _ := os.Open("file.txt")
    defer file.Close() // 函数返回前执行
    // 处理文件
}
```

## 常用函数/操作

### 字符串操作

```go
strings.Contains("hello", "ell") // true
strings.Join([]string{"a","b"}, "-") // "a-b"
strings.Split("a-b-c", "-") // ["a","b","c"]
strings.Replace("foo", "o", "0", -1) // "f00"
strconv.Itoa(123) // "123"
strconv.Atoi("123") // 123, error
```

### 切片操作

```go
len(slice) // 长度
cap(slice) // 容量
copy(dest, src) // 复制切片
make([]int, 5) // 创建切片
```

### 映射操作

```go
len(m) // 元素数量
value, exists := m["key"] // 检查存在
```

### 文件操作

```go
os.Create("file.txt") // 创建
os.Open("file.txt") // 打开
ioutil.ReadFile("file.txt") // 读取全部
ioutil.WriteFile("file.txt", data, 0644) // 写入
```

### 时间操作

```go
time.Now() // 当前时间
time.Sleep(2 * time.Second) // 休眠
t := time.Date(2023, 1, 1, 0, 0, 0, 0, time.UTC)
t.Format("2006-01-02 15:04:05") // 格式化
```

### 并发

```go
go func() { /*...*/ }() // 启动goroutine

var wg sync.WaitGroup
wg.Add(1)
go func() {
    defer wg.Done()
    // 工作代码
}()
wg.Wait()

// 通道
ch := make(chan int)
go func() { ch <- 1 }()
val := <-ch

// 带缓冲通道
ch := make(chan int, 2)
```

### JSON处理

```go
json.Marshal(data) // 编码
json.Unmarshal(jsonData, &result) // 解码
```

### 错误处理

```go
if err != nil {
    log.Fatal(err)
}

// 自定义错误
errors.New("error message")
fmt.Errorf("format error: %v", err)
```

### 反射

```go
reflect.TypeOf(x) // 获取类型
reflect.ValueOf(x) // 获取值
```

### 测试

```go
// test_test.go
func TestAdd(t *testing.T) {
    got := add(1, 2)
    want := 3
    if got != want {
        t.Errorf("got %d, want %d", got, want)
    }
}
```

### 常用包

```go
fmt, os, io, strings, strconv, 
math, time, encoding/json, 
net/http, sync, context, 
testing, reflect, errors
```