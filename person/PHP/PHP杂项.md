# PHP杂项

> 官方文档参考：<https://www.php.net/manual/zh/ref.misc.php>

| 函数名                                                                         | 说明                                                       |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| `connection_aborted` ( void ) : int                                            | 检查客户端是否已经断开                                     |
| `connection_status` ( void ) : int                                             | 返回连接的状态位                                           |
| `constant` ( string $name ) : mixed                                            | 返回一个常量的值                                           |
| `define` ( string $name , mixed $value [,bool $case_insensitive=false]) : bool | 定义一个常量                                               |
| `defined`( string $name ) : bool                                               | 检查某个名称的常量是否存在                                 |
| `eval`( string $code ) : mixed                                                 | 把字符串作为PHP代码执行                                    |
| `get_browser`([ string $user_agent [,bool $return_array = false]]) : mixed     | 获取浏览器具有的功能                                       |
| `__halt_compiler`( void ) : void                                               | 中断编译器的执行,常用于在PHP脚本内嵌入数据，类似于安装文件 |
| `highlight_file`( string $filename [,bool $return=false]) : mixed              | 语法高亮一个文件                                           |
| `highlight_string`( string $str [,bool $return=false]) : mixed                 | 字符串的语法高亮                                           |
| `hrtime`                                                                       | Get the system's high resolution time                      |
| `ignore_user_abort`([ bool $value ] ) : int                                    | 设置客户端断开连接时是否中断脚本的执行                     |
| `pack`( string $format [,mixed $args [,mixed $... ]]) : string                 | 将数据打包成二进制字符串                                   |
| `php_check_syntax`( string $filename [,string &$error_message]) : bool         | 检查PHP的语法（并执行）指定的文件                          |
| `php_strip_whitespace`( string $filename ) : string                            | 返回删除注释和空格后的PHP源码                              |
| `show_source`                                                                  | 别名 `highlight_file` ：语法高亮一个文件                   |
| `sys_getloadavg`( void ) : array                                               | 获取系统的负载（load average），1分钟、5分钟和15分钟之前   |
| `time_sleep_until`( float $timestamp ) : bool                                  | 使脚本睡眠到指定的时间为止。                               |
| `uniqid`([ string $prefix = "" [, bool $more_entropy = false ]] ) : string     | 生成一个唯一ID: 获取一个带前缀、基于当前时间微秒数的唯一ID |
| `unpack`( string $format , string $data [,int $offset=0]) : array              | 二进制字符串对数据进行解包                                 |
| `sleep`( int $seconds ) : int                                                  | 延缓执行                                                   |
| `time_nanosleep`( int $seconds , int $nanoseconds ) : mixed                    | 延缓执行若干秒和纳秒                                       |
| `usleep`( int $micro_seconds ) : void                                          | 以指定的微秒数延迟执行                                     |

## 1、冲刷(flush)所有响应的数据给客户端并结束请求

```php
if (!function_exists("fastcgi_finish_request")) {  // FastCGI模式可用
    function fastcgi_finish_request()  {
    }
}
fastcgi_finish_request();
set_time_limit(0);//避免超时
ini_set('memory_limit','-1'); //避免内存不足
//后台自行执行的业务逻辑

```

## 2、客户端断开连接时中断脚本继续执行

```php
ignore_user_abort(true);
```
