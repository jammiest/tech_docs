# PHP内置函数速查手册完整版

## 第一部分：常用函数

### 字符串处理
```php
strlen($string)             # 返回字符串长度
substr($string,$start,$len) # 截取字符串部分
str_replace($search,$replace,$subject) # 字符串替换
trim($string,$charlist)     # 去除首尾空白或指定字符
strtolower($string)         # 转换为小写
strtoupper($string)         # 转换为大写
strpos($haystack,$needle)   # 查找字符串首次出现位置
explode($delimiter,$string) # 字符串分割为数组
implode($glue,$pieces)      # 数组组合为字符串
str_split($string,$length)  # 将字符串分割为数组
```

### 数组处理
```php
array()                     # 创建数组
count($array)               # 计算数组元素个数
array_push($array,$value)   # 向数组末尾添加元素
array_pop($array)           # 弹出数组最后一个元素
array_merge($array1,$array2) # 合并数组
array_keys($array)          # 返回所有键名
array_values($array)        # 返回所有值
in_array($needle,$haystack) # 检查值是否存在
array_search($needle,$haystack) # 搜索给定值
sort($array)                # 对数组升序排序
```

### 数学运算
```php
abs($number)                # 绝对值
ceil($number)               # 向上取整
floor($number)              # 向下取整
round($number,$precision)   # 四舍五入
max($values)                # 找出最大值
min($values)                # 找出最小值
rand($min,$max)             # 生成随机数
sqrt($number)               # 平方根
pow($base,$exp)             # 指数表达式
mt_rand($min,$max)          # 生成更好的随机数
```

### 日期时间
```php
date($format,$timestamp)    # 格式化日期/时间
time()                      # 返回当前Unix时间戳
strtotime($time)            # 将字符串转为时间戳
mktime($hour,$min,$sec,$month,$day,$year) # 取得日期时间戳
date_diff($datetime1,$datetime2) # 计算日期差
microtime($as_float)        # 返回当前Unix时间戳和微秒数
date_default_timezone_set($timezone) # 设置默认时区
```

### 文件系统
```php
file_exists($filename)      # 检查文件/目录是否存在
file_get_contents($filename) # 读取文件内容到字符串
file_put_contents($filename,$data) # 写入字符串到文件
is_dir($path)               # 判断是否是目录
mkdir($pathname)            # 创建目录
scandir($directory)         # 列出目录内容
filemtime($filename)        # 获取文件修改时间
filesize($filename)         # 获取文件大小
pathinfo($path)             # 返回路径信息
realpath($path)             # 返回规范化的绝对路径
```

## 第二部分：更多函数

### 字符串处理
```php
strrev($string)             # 反转字符串
str_repeat($input,$multiplier) # 重复字符串
str_pad($input,$length,$pad_string,$pad_type) # 填充字符串
str_shuffle($string)        # 随机打乱字符串
wordwrap($string,$width,$break,$cut) # 打断字符串为指定数量的字串
str_word_count($string,$format,$charlist) # 返回字符串中单词的使用情况
levenshtein($str1,$str2)    # 计算两个字符串的编辑距离
similar_text($str1,$str2,$percent) # 计算两个字符串的相似度
htmlspecialchars($string)   # 将特殊字符转换为HTML实体
htmlentities($string)       # 将所有字符转换为HTML实体
```

### 数组处理
```php
array_chunk($array,$size,$preserve_keys) # 分割数组
array_fill($start_index,$num,$value) # 用值填充数组
array_filter($array,$callback) # 用回调函数过滤数组元素
array_flip($array)         # 交换键和值
array_map($callback,$array) # 为数组每个元素应用回调
array_reduce($array,$callback,$initial) # 用回调迭代地将数组简化为单一值
array_reverse($array,$preserve_keys) # 返回单元顺序相反的数组
array_slice($array,$offset,$length,$preserve_keys) # 从数组中取出一段
array_unique($array,$sort_flags) # 移除数组中重复的值
array_walk($array,$callback,$userdata) # 对数组每个成员应用用户函数
```

### 数学运算
```php
base_convert($number,$frombase,$tobase) # 在任意进制之间转换数字
bindec($binary_string)     # 二进制转十进制
decbin($number)            # 十进制转二进制
dechex($number)            # 十进制转十六进制
hexdec($hex_string)        # 十六进制转十进制
deg2rad($number)           # 将角度转换为弧度
rad2deg($number)           # 将弧度转换为角度
hypot($x,$y)               # 计算直角三角形的斜边长度
log($arg,$base)            # 自然对数
pi()                       # 得到圆周率值
```

### 日期时间
```php
checkdate($month,$day,$year) # 验证格里高里日期
date_add($object,$interval) # 增加日期间隔
date_create($time,$timezone) # 创建新的DateTime对象
date_format($object,$format) # 格式化日期
date_parse($date)          # 返回日期的详细信息
date_sub($object,$interval) # 减去日期间隔
getdate($timestamp)        # 获取日期/时间信息
gmdate($format,$timestamp) # 格式化GMT/UTC日期/时间
idate($format,$timestamp)  # 将本地时间/日期格式化为整数
localtime($timestamp,$is_associative) # 返回本地时间
```

### 文件系统
```php
basename($path,$suffix)    # 返回路径中的文件名部分
dirname($path,$levels)     # 返回路径中的目录部分
fileatime($filename)       # 获取文件的上次访问时间
filectime($filename)       # 获取文件的inode修改时间
filegroup($filename)       # 取得文件的组
fileowner($filename)       # 取得文件的所有者
fileperms($filename)       # 取得文件的权限
fopen($filename,$mode)     # 打开文件或URL
fclose($handle)            # 关闭打开的文件指针
fread($handle,$length)      # 读取文件（二进制安全）
```

### 数据库操作
```php
mysqli_connect($host,$user,$pass,$db) # 连接MySQL数据库
mysqli_query($link,$query)   # 执行SQL查询
mysqli_fetch_assoc($result)  # 获取关联数组结果
mysqli_num_rows($result)     # 获取结果行数
mysqli_insert_id($link)     # 获取最后插入ID
mysqli_close($link)         # 关闭连接
PDO::__construct($dsn,$user,$pass) # 创建PDO实例
PDO::query($statement)       # 执行SQL语句
PDOStatement::fetch($style)  # 从结果集中获取下一行
```

### 正则表达式
```php
preg_match($pattern,$subject,$matches) # 执行正则匹配
preg_replace($pattern,$replacement,$subject) # 执行正则替换
preg_split($pattern,$subject) # 通过正则分割字符串
preg_match_all($pattern,$subject,$matches) # 执行全局正则匹配
preg_grep($pattern,$input)   # 返回匹配模式的数组条目
preg_last_error()            # 返回最后发生的PCRE错误
preg_quote($str,$delimiter)  # 转义正则表达式字符
```

### 会话控制
```php
session_start()             # 启动新会话
session_id($id)             # 获取/设置会话ID
session_destroy()           # 销毁会话数据
session_regenerate_id()     # 更新会话ID
session_unset()             # 释放所有会话变量
session_name($name)         # 读取/设置会话名称
session_save_path($path)    # 读取/设置会话保存路径
session_status()            # 返回当前会话状态
```

### 密码安全
```php
password_hash($password,$algo) # 创建密码哈希
password_verify($password,$hash) # 验证密码匹配哈希
password_needs_rehash($hash,$algo) # 检查是否需要重新哈希
crypt($str,$salt)          # 单向字符串散列
hash($algo,$data)          # 生成哈希值
hash_hmac($algo,$data,$key) # 使用HMAC方法生成哈希
openssl_encrypt($data,$method,$key) # 加密数据
openssl_decrypt($data,$method,$key) # 解密数据
random_bytes($length)      # 生成密码学安全的随机字节
random_int($min,$max)      # 生成密码学安全的随机整数
```

### 图像处理(GD库)
```php
imagecreate($width,$height) # 创建新图像
imagecolorallocate($image,$red,$green,$blue) # 为图像分配颜色
imagefill($image,$x,$y,$color) # 区域填充
imageline($image,$x1,$y1,$x2,$y2,$color) # 画线
imagerectangle($image,$x1,$y1,$x2,$y2,$color) # 画矩形
imageellipse($image,$cx,$cy,$width,$height,$color) # 画椭圆
imagepng($image,$filename) # 输出PNG图像
imagejpeg($image,$filename,$quality) # 输出JPEG图像
imagegif($image,$filename) # 输出GIF图像
imagedestroy($image)      # 销毁图像资源
```

### XML处理
```php
simplexml_load_string($data) # 将XML字符串解析为对象
simplexml_load_file($file)   # 将XML文档解析为对象
DOMDocument::load($file)     # 从文件加载XML
DOMDocument::save($file)     # 将XML保存到文件
xml_parser_create($encoding) # 创建XML解析器
xml_parse($parser,$data,$is_final) # 解析XML文档
xml_parser_free($parser)     # 释放XML解析器
```

### 压缩函数
```php
gzcompress($data,$level)     # 压缩字符串
gzuncompress($data)         # 解压缩字符串
gzencode($data,$level)      # 创建gzip压缩字符串
gzdecode($data)             # 解压缩gzip字符串
gzdeflate($data,$level)     # 使用DEFLATE算法压缩
gzinflate($data,$length)    # 解压缩DEFLATE压缩数据
```

### 网络函数
```php
gethostbyname($hostname)     # 返回主机名对应的IP
gethostbynamel($hostname)    # 返回主机名对应的IP列表
inet_pton($address)         # 将IP地址转换为二进制格式
inet_ntop($in_addr)         # 将二进制IP转换为可读格式
fsockopen($hostname,$port,$errno,$errstr,$timeout) # 打开网络连接
pfsockopen($hostname,$port,$errno,$errstr,$timeout) # 打开持久网络连接
socket_create($domain,$type,$protocol) # 创建套接字
socket_connect($socket,$address,$port) # 发起套接字连接
```

### 系统执行
```php
exec($command,$output,$return_var) # 执行外部程序
shell_exec($command)         # 通过shell执行命令
system($command,$return_var)  # 执行外部程序并显示输出
passthru($command,$return_var) # 执行外部程序并显示原始输出
proc_open($command,$descriptorspec,$pipes,$cwd,$env,$options) # 执行命令并打开文件指针
proc_close($process)         # 关闭由proc_open打开的进程
```

### 调试函数
```php
var_dump($expression)       # 打印变量的详细信息
print_r($expression,$return) # 打印变量的易读信息
debug_backtrace($options,$limit) # 产生回溯跟踪
debug_print_backtrace($options,$limit) # 打印回溯跟踪
error_log($message,$type,$destination,$headers) # 发送错误信息到日志
error_reporting($level)      # 设置错误报告级别
set_error_handler($handler)  # 设置用户自定义错误处理函数
restore_error_handler()      # 恢复之前的错误处理函数
```

### 魔术方法
```php
__construct()               # 构造函数
__destruct()               # 析构函数
__call($name,$arguments)   # 调用不可访问方法时调用
__callStatic($name,$arguments) # 调用不可访问静态方法时调用
__get($name)               # 读取不可访问属性时调用
__set($name,$value)        # 写入不可访问属性时调用
__isset($name)             # 对不可访问属性调用isset()或empty()时调用
__unset($name)             # 对不可访问属性调用unset()时调用
__sleep()                  # 序列化时调用
__wakeup()                 # 反序列化时调用
```

### SPL数据结构
```php
SplFixedArray::fromArray($array) # 从PHP数组创建固定数组
SplStack::push($value)       # 向堆栈添加元素
SplQueue::enqueue($value)    # 向队列添加元素
SplHeap::insert($value)      # 向堆插入元素
SplPriorityQueue::insert($value,$priority) # 向优先队列插入元素
SplObjectStorage::attach($object,$data) # 添加对象到存储
SplFileInfo::getRealPath()   # 获取文件的绝对路径
SplFileObject::fgetcsv($delimiter,$enclosure,$escape) # 获取CSV行
```

### 国际化函数
```php
NumberFormatter::format($value) # 格式化数字
MessageFormatter::formatMessage($locale,$pattern,$args) # 快速格式化消息
Locale::acceptFromHttp($http_accept_language) # 从HTTP头获取最合适的区域设置
Normalizer::normalize($input,$form) # 规范化提供的输入
IntlDateFormatter::format($timestamp) # 格式化日期/时间
Collator::compare($str1,$str2) # 比较两个Unicode字符串
```

### 反射API
```php
ReflectionClass::getMethods() # 获取类的方法数组
ReflectionFunction::getParameters() # 获取函数的参数
ReflectionParameter::getType() # 获取参数的类型
ReflectionProperty::getValue($object) # 获取属性的值
ReflectionExtension::getClasses() # 获取扩展的类
ReflectionZendExtension::getVersion() # 获取Zend扩展的版本
```

### 多进程控制
```php
pcntl_fork()                # 创建子进程
pcntl_waitpid($pid,$status,$options) # 等待或返回fork的子进程状态
pcntl_signal($signo,$handler,$restart_syscalls) # 安装信号处理器
posix_getpid()              # 返回当前进程ID
posix_getppid()             # 返回父进程ID
posix_kill($pid,$sig)       # 向进程发送信号
```

### 共享内存
```php
shmop_open($key,$flags,$mode,$size) # 创建/打开共享内存块
shmop_read($shmid,$start,$count) # 从共享内存读取
shmop_write($shmid,$data,$offset) # 向共享内存写入
shmop_delete($shmid)        # 删除共享内存块
shmop_close($shmid)         # 关闭共享内存块
sem_get($key,$max_acquire,$perm,$auto_release) # 获取信号量标识
sem_acquire($sem_identifier) # 获取信号量
sem_release($sem_identifier) # 释放信号量
```

### 其他扩展函数
```php
mb_strlen($str,$encoding)    # 获取字符串长度(多字节)
mb_substr($str,$start,$len,$encoding) # 获取字符串部分(多字节)
iconv($in_charset,$out_charset,$str) # 转换字符编码
filter_var($variable,$filter,$options) # 过滤变量
get_defined_constants($categorize) # 返回所有已定义常量
get_defined_functions()    # 返回所有已定义函数
get_defined_vars()         # 返回所有已定义变量
get_resource_type($handle)  # 返回资源类型
```