# musl与glibc功能对比

## 标准I/O实现

- **printf行为**：
  - glibc支持一些非标准格式说明符(如%Ld作为%lld的别名)
  - musl不支持这些非标准格式说明符，遇到时会返回错误
  - musl不支持glibc的自定义格式说明符注册系统
  - musl的浮点数转换遵循舍入模式，glibc不遵循

- **文件结束行为**：
  - musl遵循ISO C和POSIX要求，EOF状态是粘性的
  - glibc 2.28之前版本会忽略EOF状态，如果有新输入仍会返回

- **读写模式**：
  - musl使用readv和writev实现stdio
  - 读写操作更高效，不依赖启发式判断何时缓冲或传输

## 信号掩码和setjmp/longjmp

- glibc默认保存和恢复信号掩码作为jmp_buf的一部分
- musl默认不包含信号掩码，除非使用sigsetjmp/siglongjmp明确要求
- musl的行为为正确使用接口的应用提供了更好的性能

## 正则表达式

- musl基于TRE实现，但做了重大修改
- 不支持GNU regex的替代API
- 在64位架构上，musl使用正确的regoff_t类型，与glibc不兼容

## exit的可重入性

- musl不支持递归调用exit(通过atexit函数/析构函数)或多线程同时调用exit
- glibc在递归exit时不会死锁，但不清楚是否是有意支持

## 动态链接器

- **延迟绑定**：
  - glibc支持延迟绑定
  - musl不支持延迟绑定，改为"延迟绑定"机制

- **卸载库**：
  - musl的dlclose是空操作，库会一直加载到进程退出
  - glibc会引用计数并在计数归零时卸载库

- **dlerror线程安全**：
  - musl 1.1.9+和glibc都使用线程本地错误缓冲区
  - 早期musl版本使用全局缓冲区

## 线程

- **栈大小**：
  - glibc默认2-10MB
  - musl默认128k(1.1.21+)，之前是80k

- **取消**：
  - musl提供POSIX指定的取消副作用保证
  - glibc没有此类保证

## 字符集和区域设置

- **C区域设置**：
  - musl 1.1.11+提供特殊C区域设置，将0x80-0xff视为抽象单字节字符
  - glibc有伪ASCII、伪8位C区域设置

- **默认区域设置**：
  - glibc默认"C"
  - musl默认"C.UTF-8"

- **UTF-8定义**：
  - musl遵循Unicode标准，拒绝代理项、过长序列和5/6字节序列
  - glibc接受5和6字节序列

## 名称解析/DNS

- musl并行查询所有nameserver，接受最先到达的响应
- glibc顺序尝试每个nameserver
- musl 1.1.13+支持resolv.conf中的domain和search关键字，但行为略有不同

## 其他差异

- **getopt**：
  - GNU getopt重排argv将选项前置
  - musl和POSIX getopt在第一个非选项参数处停止处理

- **basename**：
  - glibc提供两个版本
  - musl只提供标准版本

- **system**：
  - glibc提供线程安全版本
  - musl版本不是线程安全的