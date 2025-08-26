# Go 语言标准库常用核心函数/方法速查手册

## 一、输入输出 (I/O)

### fmt - 格式化 I/O
```go
Print(a ...interface{})                 // 将参数写入标准输出，无格式
Println(a ...interface{})               // 将参数写入标准输出，以空格分隔并在末尾添加换行符
Printf(format string, a ...interface{}) // 根据格式说明符格式化并写入标准输出
Sprintf(format string, a ...interface{}) string // 格式化并返回字符串
Errorf(format string, a ...interface{}) error   // 格式化并返回一个 error 类型
Scan(a ...interface{})                  // 从标准输入扫描文本，将空格分隔的值存入参数
Scanf(format string, a ...interface{})  // 根据格式字符串从标准输入扫描文本
Scanln(a ...interface{})                // 从标准输入扫描一行文本，直到换行
Fprintf(w io.Writer, format string, a ...interface{}) (int, error) // 格式化并写入指定的 io.Writer
```
### os - 操作系统交互
```go
Open(name string) (*File, error)        // 以只读模式打开一个文件
Create(name string) (*File, error)      // 创建或截断一个文件（只写模式）
Remove(name string) error               // 删除文件或空目录
Getwd() (dir string, err error)         // 获取当前工作目录
Exit(code int)                          // 以指定的状态码退出程序
```
### io/ioutil - 实用 I/O 函数 (Go 1.16+ 后部分函数移至 os 和 io)
```go
ReadFile(filename string) ([]byte, error) // 读取整个文件的内容
WriteFile(filename string, data []byte, perm os.FileMode) error // 将数据写入文件
ReadAll(r io.Reader) ([]byte, error)     // 从 io.Reader 读取直到错误或 EOF
```
### bufio - 带缓冲的 I/O
```go
NewReader(rd io.Reader) *Reader         // 创建一个带缓冲的 Reader
NewScanner(r io.Reader) *Scanner        // 创建一个 Scanner，常用于逐行读取输入
(*Scanner) Scan() bool                  // 扫描下一行或下一个token，到达尾部返回false
(*Scanner) Text() string                // 返回 Scan() 方法最新扫描到的文本
NewWriter(w io.Writer) *Writer          // 创建一个带缓冲的 Writer
(*Writer) WriteString(s string) (int, error) // 写入一个字符串
(*Writer) Flush() error                 // 将缓冲中的数据刷到底层的 io.Writer
```

---

## 二、字符串处理 (strings, strconv)

### strings - 字符串操作
```go
Contains(s, substr string) bool         // 判断字符串 s 是否包含子串 substr
HasPrefix(s, prefix string) bool        // 判断字符串 s 是否以 prefix 开头
HasSuffix(s, suffix string) bool        // 判断字符串 s 是否以 suffix 结尾
Index(s, substr string) int             // 返回子串 substr 在字符串 s 中第一次出现的位置
Join(elems []string, sep string) string // 将字符串切片连接为一个字符串，用 sep 分隔
Split(s, sep string) []string           // 将字符串 s 按分隔符 sep 分割成切片
Replace(s, old, new string, n int) string // 将字符串 s 中的前 n 个 old 替换为 new
ToLower(s string) string                // 将字符串转换为小写
ToUpper(s string) string                // 将字符串转换为大写
Trim(s, cutset string) string           // 去除字符串 s 首尾连续的包含在 cutset 中的字符
Fields(s string) []string               // 按一个或多个连续的空格分割字符串
```
### strconv - 字符串与基本数据类型转换
```go
Atoi(s string) (int, error)             // 字符串转 int
Itoa(i int) string                      // int 转字符串
ParseInt(s string, base int, bitSize int) (int64, error) // 字符串解析为指定进制的整数
ParseFloat(s string, bitSize int) (float64, error) // 字符串解析为浮点数
FormatInt(i int64, base int) string     // 整数格式化为指定进制的字符串
FormatFloat(f float64, fmt byte, prec, bitSize int) string // 浮点数格式化为字符串
```

---

## 三、集合操作 (slices, maps)

### slices - 切片操作 (Go 1.21+)
```go
Contains(s []T, v T) bool               // 判断切片 s 是否包含元素 v
Compare(s1, s2 []T) int                 // 比较两个切片，元素依次比较
Index(s []T, v T) int                   // 返回元素 v 在切片 s 中第一次出现的索引
Insert(s []T, i int, v ...T) []T        // 在索引 i 处插入元素 v
Delete(s []T, i, j int) []T             // 删除切片 s 中索引从 i 到 j-1 的元素
Clone(s []T) []T                        // 创建切片 s 的副本
Sort(s []T)                             // 对切片进行排序（需要切片元素有序）
```

---

## 四、时间与日期 (time)

### time - 时间操作
```go
Now() Time                               // 获取当前本地时间
Date(year, month, day, hour, min, sec, nsec int, loc *Location) Time // 构造一个时间
Parse(layout, value string) (Time, error) // 根据格式字符串解析时间
(Time) Format(layout string) string      // 将时间格式化为字符串
(Time) Add(d Duration) Time              // 时间加上一个时间段
Since(t Time) Duration                   // 从时间 t 到现在经过的时间
Sleep(d Duration)                        // 让当前 Goroutine 休眠一段时间段
Tick(d Duration) <-chan Time             // 返回一个通道，它每隔一段时间 d 发送一个时间点
```

---

## 五、数学运算 (math, math/rand)

### math - 数学常数和函数
```go
Pi float64                               // 数学常数 π
Max(x, y float64) float64               // 返回 x 和 y 中的最大值
Min(x, y float64) float64               // 返回 x 和 y 中的最小值
Pow(x, y float64) float64               // 返回 x 的 y 次幂
Sqrt(x float64) float64                 // 返回 x 的平方根
Ceil(x float64) float64                 // 返回大于等于 x 的最小整数（向上取整）
Floor(x float64) float64                // 返回小于等于 x 的最大整数（向下取整）
Round(x float64) float64                // 对 x 进行四舍五入
```
### math/rand - 伪随机数生成
```go
Intn(n int) int                         // 返回一个 [0, n) 之间的随机整数
Seed(seed int64)                        // 初始化随机数生成器的种子（通常用 time.Now().UnixNano()）
Float64() float64                       // 返回一个 [0.0, 1.0) 之间的随机浮点数
Shuffle(n int, swap func(i, j int))     // 随机打乱指定数量的元素顺序
```

---

## 六、编码/解码 (encoding/json)

### encoding/json - JSON 处理
```go
Marshal(v interface{}) ([]byte, error)  // 将 Go 对象 v 序列化为 JSON 字节数组
Unmarshal(data []byte, v interface{}) error // 将 JSON 字节数组反序列化到 Go 对象 v 中
NewEncoder(w io.Writer) *Encoder        // 创建一个写入 io.Writer 的 JSON 编码器
(*Encoder) Encode(v interface{}) error  // 将对象编码为 JSON 并写入流
NewDecoder(r io.Reader) *Decoder        // 创建一个从 io.Reader 读取的 JSON 解码器
(*Decoder) Decode(v interface{}) error  // 从流中读取并解码 JSON 到对象
```

---

## 七、并发编程 (sync)

### sync - 同步原语
```go
WaitGroup                               // 用于等待一组 Goroutine 结束
    (wg *WaitGroup) Add(delta int)      // 增加等待的 Goroutine 数量
    (wg *WaitGroup) Done()              // 表示一个 Goroutine 已完成（等价于 Add(-1)）
    (wg *WaitGroup) Wait()              // 阻塞直到所有 Goroutine 都调用了 Done
Mutex                                   // 互斥锁
    (m *Mutex) Lock()                   // 获取锁
    (m *Mutex) Unlock()                 // 释放锁
RWMutex                                 // 读写互斥锁
    (m *RWMutex) RLock()                // 获取读锁
    (m *RWMutex) RUnlock()              // 释放读锁
Once                                    // 确保某个操作只执行一次
    (o *Once) Do(f func())              // 执行函数 f，且只执行一次
```

---

## 八、网络编程 (net/http)

### net/http - HTTP 客户端和服务端
```go
Get(url string) (*Response, error)      // 发送 HTTP GET 请求
Post(url, contentType string, body io.Reader) (*Response, error) // 发送 HTTP POST 请求
HandleFunc(pattern string, handler func(ResponseWriter, *Request)) // 注册处理函数
ListenAndServe(addr string, handler Handler) error // 启动一个 HTTP 服务器
Error(w ResponseWriter, error string, code int) // 返回一个错误响应
NewRequest(method, url string, body io.Reader) (*Request, error) // 创建一个请求
```