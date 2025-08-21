# PHP知识点速记

## 一、基础语法

1. **PHP 标记**
```php
<?php // 标准开始标记
?>   // 结束标记（纯PHP文件可省略）
<?= $variable ?> // 简短输出标记
```

2. **注释**
```php
// 单行注释
# 另一种单行注释
/*
多行
注释
*/
```

## 二、变量与常量

1. **变量**
```php
$var = 'value';          // 区分大小写
$camelCase = '变量名';    // 命名规范
unset($var);             // 销毁变量
isset($var);             // 检测变量是否设置
empty($var);             // 检测变量是否为空
```

2. **变量作用域**
```php
$globalVar = 'global';   // 全局变量

function test() {
    global $globalVar;    // 访问全局变量
    static $staticVar = 0; // 静态变量
    $localVar = 'local';  // 局部变量
}
```

3. **超全局变量**
```php
$_GET       // GET参数
$_POST      // POST数据
$_REQUEST   // GET+POST+COOKIE
$_SERVER    // 服务器信息
$_SESSION   // 会话数据
$_COOKIE    // Cookie数据
$_FILES     // 上传文件
$_ENV       // 环境变量
$GLOBALS    // 全局变量数组
```

4. **常量**
```php
define('CONSTANT', 'value'); // 定义常量
const CONSTANT2 = 'value';   // PHP 5.3+
defined('CONSTANT');         // 检查常量
```

## 三、数据类型

1. **标量类型**
```php
$int = 123;         // 整型
$float = 3.14;      // 浮点型
$string = "hello";  // 字符串
$bool = true;       // 布尔型
```

2. **复合类型**
```php
$array = [1, 2, 3];             // 数组
$object = new stdClass();       // 对象
$callable = function() {};      // 可调用
```

3. **特殊类型**
```php
$null = null;       // NULL
$resource = fopen('file', 'r'); // 资源
```

4. **类型检测**
```php
is_int($var); is_float($var); is_string($var);
is_bool($var); is_array($var); is_object($var);
is_callable($var); is_resource($var); is_null($var);
```

5. **类型转换**
```php
(int)$var; (float)$var; (string)$var;
(bool)$var; (array)$var; (object)$var;
settype($var, 'string'); // 永久转换
```

## 四、运算符

1. **算术运算符**
```php
+ - * / % **
```

2. **赋值运算符**
```php
= += -= *= /= %= .= **=
```

3. **比较运算符**
```php
== != === !== <> < > <= >= <=>
```

4. **逻辑运算符**
```php
&& || ! and or xor
```

5. **其他运算符**
```php
.       // 字符串连接
?:      // 三元运算符
??      // NULL合并运算符(PHP7+)
@       // 错误抑制
instanceof // 对象类型检查
```

## 五、流程控制

1. **条件语句**
```php
if (condition) {
} elseif (condition) {
} else {
}

switch ($var) {
    case 'value':
        break;
    default:
}
```

2. **循环语句**
```php
while (condition) {
}

do {
} while (condition);

for ($i=0; $i<10; $i++) {
}

foreach ($array as $value) {
}

foreach ($array as $key => $value) {
}
```

3. **流程控制关键字**
```php
break;      // 退出循环
continue;   // 跳过本次循环
return;     // 函数返回
exit;       // 终止脚本
die;        // 同exit
```

## 六、函数

1. **函数定义**
```php
function funcName($param1, $param2 = 'default') {
    return $param1 + $param2;
}
```

2. **参数类型声明(PHP7+)**
```php
function func(int $param, string $param2): bool {
    return true;
}
```

3. **可变参数**
```php
function sum(...$numbers) {
    return array_sum($numbers);
}
```

4. **匿名函数**
```php
$func = function($param) {
    return $param * 2;
};
```

5. **箭头函数(PHP7.4+)**
```php
$func = fn($x) => $x * 2;
```

## 七、数组

1. **数组创建**
```php
$arr1 = array(1, 2, 3);
$arr2 = [1, 2, 3]; // 简写
$assoc = ['name'=>'John', 'age'=>30];
```

2. **数组操作**
```php
$arr[] = 'new';     // 添加元素
count($arr);        // 元素数量
array_push($arr, 'a', 'b'); // 末尾添加
array_pop($arr);    // 移除末尾
array_shift($arr);  // 移除开头
array_unshift($arr, 'a'); // 开头添加
```

3. **数组遍历**
```php
foreach ($arr as $value) {}
foreach ($arr as $key => $value) {}
```

4. **数组排序**
```php
sort($arr);     // 值升序
rsort($arr);    // 值降序
asort($arr);    // 保持键值关联升序
arsort($arr);   // 保持键值关联降序
ksort($arr);    // 按键名升序
krsort($arr);   // 按键名降序
```

5. **数组函数**
```php
array_merge($arr1, $arr2); // 合并
array_slice($arr, 0, 2);   // 截取
array_splice($arr, 0, 1);  // 删除并替换
array_search('value', $arr); // 搜索
in_array('value', $arr);    // 检查存在
array_key_exists('key', $arr); // 检查键
array_reverse($arr);       // 反转
array_unique($arr);        // 去重
```

## 八、字符串

1. **字符串定义**
```php
$str1 = '单引号';
$str2 = "双引号"; // 可解析变量
$str3 = <<<EOD
heredoc语法
EOD;
```

2. **字符串操作**
```php
$str1 . $str2;      // 连接
strlen($str);       // 长度
substr($str, 0, 5); // 截取
str_replace('old', 'new', $str); // 替换
trim($str);         // 去除空白
strtolower($str);   // 转小写
strtoupper($str);   // 转大写
ucfirst($str);      // 首字母大写
ucwords($str);      // 每个单词首字母大写
```

3. **字符串查找**
```php
strpos($str, 'sub');    // 首次出现位置
stripos($str, 'sub');   // 不区分大小写
strrpos($str, 'sub');   // 最后出现位置
strstr($str, 'sub');    // 首次出现后的字符串
stristr($str, 'sub');   // 不区分大小写
```

4. **字符串分割与组合**
```php
explode(',', $str); // 分割为数组
implode(',', $arr); // 数组组合为字符串
str_split($str, 2); // 按长度分割
```

5. **字符串格式化**
```php
sprintf("格式 %s %d", $str, $num); // 格式化
printf("格式 %s", $str);           // 输出格式化
```

## 九、面向对象

1. **类定义**
```php
class MyClass {
    // 属性
    public $publicProp;
    protected $protectedProp;
    private $privateProp;
    
    // 常量
    const CLASS_CONST = 'value';
    
    // 方法
    public function __construct($param) {
        $this->publicProp = $param;
    }
    
    public function publicMethod() {}
    
    protected function protectedMethod() {}
    
    private function privateMethod() {}
    
    // 静态方法
    public static function staticMethod() {}
    
    // 魔术方法
    public function __toString() {
        return 'string representation';
    }
}
```

2. **继承与多态**
```php
class ChildClass extends ParentClass {
    public function __construct($param) {
        parent::__construct($param);
    }
    
    // 方法重写
    public function publicMethod() {
        // 新实现
    }
}
```

3. **接口与抽象类**
```php
interface MyInterface {
    public function interfaceMethod();
}

abstract class AbstractClass {
    abstract public function abstractMethod();
    
    public function concreteMethod() {
        // 具体实现
    }
}
```

4. **特性(Trait)**
```php
trait MyTrait {
    public function traitMethod() {
        // 实现
    }
}

class MyClass {
    use MyTrait;
}
```

5. **对象操作**
```php
$obj = new MyClass('param'); // 实例化
$obj->publicProp;            // 访问属性
$obj->publicMethod();        // 调用方法
MyClass::staticMethod();      // 调用静态方法
MyClass::CLASS_CONST;        // 访问常量
$obj instanceof MyClass;     // 类型检查
```

## 十、错误处理

1. **错误级别**
```php
E_ERROR       // 致命运行时错误
E_WARNING     // 运行时警告
E_PARSE       // 编译时解析错误
E_NOTICE      // 运行时通知
E_STRICT      // 编码标准化警告
E_DEPRECATED  // 未来版本可能不工作的警告
```

2. **错误处理**
```php
error_reporting(E_ALL);  // 设置报告级别
ini_set('display_errors', 1); // 显示错误
```

3. **自定义错误处理**
```php
set_error_handler(function($errno, $errstr) {
    // 自定义处理
});

set_exception_handler(function($exception) {
    // 异常处理
});

register_shutdown_function(function() {
    // 脚本结束处理
});
```

4. **异常处理**
```php
try {
    // 可能抛出异常的代码
    throw new Exception('message', 123);
} catch (Exception $e) {
    echo $e->getMessage();
    echo $e->getCode();
} finally {
    // 最终执行
}
```

## 十一、文件系统

1. **文件操作**
```php
file_exists('file.txt');     // 检查存在
is_file('file.txt');         // 检查是文件
is_dir('directory');        // 检查是目录
filesize('file.txt');       // 文件大小
filemtime('file.txt');      // 修改时间

// 读写文件
file_get_contents('file.txt');
file_put_contents('file.txt', $data);

// 文件指针操作
$handle = fopen('file.txt', 'r');
fread($handle, 1024);
fwrite($handle, $data);
fclose($handle);
```

2. **目录操作**
```php
mkdir('new_dir');           // 创建目录
rmdir('empty_dir');         // 删除目录
scandir('directory');       // 列出目录内容
glob('*.txt');              // 模式匹配文件
```

3. **文件上传**
```php
// HTML表单需设置 enctype="multipart/form-data"
move_uploaded_file($_FILES['file']['tmp_name'], 'path/to/file');
```

## 十二、会话与Cookie

1. **Session**
```php
session_start();            // 启动会话
$_SESSION['key'] = 'value'; // 设置会话变量
unset($_SESSION['key']);    // 删除会话变量
session_destroy();          // 销毁会话
```

2. **Cookie**
```php
setcookie('name', 'value', time()+3600, '/'); // 设置
$_COOKIE['name'];           // 读取
setcookie('name', '', time()-3600); // 删除
```

## 十三、常用函数

1. **数学函数**
```php
abs(-5);            // 绝对值
ceil(4.3);          // 向上取整
floor(4.7);         // 向下取整
round(4.5);         // 四舍五入
rand(1, 100);       // 随机数
min(1, 2, 3);       // 最小值
max(1, 2, 3);       // 最大值
```

2. **日期时间**
```php
date('Y-m-d H:i:s'); // 格式化日期
time();             // 当前时间戳
strtotime('+1 day'); // 字符串转时间戳
mktime(0, 0, 0, 1, 1, 2020); // 生成时间戳
```

3. **JSON处理**
```php
json_encode($data); // 编码为JSON
json_decode($json); // 解码JSON
json_last_error();  // 获取最后错误
```

4. **URL处理**
```php
urlencode('string'); // URL编码
urldecode('string'); // URL解码
parse_url('url');    // 解析URL
http_build_query(['a'=>1]); // 生成查询字符串
```

5. **其他实用函数**
```php
filter_var('test@example.com', FILTER_VALIDATE_EMAIL); // 验证
gettype($var);      // 获取类型
var_dump($var);     // 打印变量信息
print_r($var);      // 打印易读信息
serialize($var);    // 序列化
unserialize($data); // 反序列化
```

## 十四、数据库(MySQLi)

1. **连接数据库**
```php
$conn = new mysqli('host', 'user', 'pass', 'dbname');
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
}
```

2. **执行查询**
```php
$result = $conn->query("SELECT * FROM table");
while ($row = $result->fetch_assoc()) {
    echo $row['column'];
}
$result->free(); // 释放结果集
```

3. **预处理语句**
```php
$stmt = $conn->prepare("INSERT INTO table (col1) VALUES (?)");
$stmt->bind_param("s", $value); // s=string, i=integer, d=double, b=blob
$stmt->execute();
$stmt->close();
```

4. **关闭连接**
```php
$conn->close();
```

## 十五、安全相关

1. **输入过滤**
```php
filter_input(INPUT_GET, 'var', FILTER_SANITIZE_STRING);
htmlspecialchars($input); // 转义HTML
strip_tags($input);      // 去除HTML标签
```

2. **密码哈希**
```php
password_hash('password', PASSWORD_DEFAULT); // 创建哈希
password_verify('password', $hash);          // 验证密码
```

3. **防止SQL注入**
```php
// 使用预处理语句
$stmt = $conn->prepare("SELECT * FROM users WHERE username = ?");
$stmt->bind_param("s", $username);
```

4. **CSRF防护**
```php
// 生成令牌
$_SESSION['token'] = bin2hex(random_bytes(32));

// 表单中
<input type="hidden" name="token" value="<?php echo $_SESSION['token']; ?>">

// 验证
if ($_POST['token'] !== $_SESSION['token']) {
    die('CSRF验证失败');
}
```

## 十六、PHP新特性(PHP7+)

1. **标量类型声明**
```php
function sum(int $a, int $b): int {
    return $a + $b;
}
```

2. **空合并运算符**
```php
$value = $a ?? $b ?? 'default';
```

3. **飞船运算符**
```php
$result = $a <=> $b; // -1, 0, 1
```

4. **匿名类**
```php
$obj = new class {
    public function method() {
        return "Anonymous";
    }
};
```

5. **生成器委托(PHP7+)**
```php
function generator() {
    yield from anotherGenerator();
}
```

6. **Null合并赋值运算符(PHP7.4+)**
```php
$array['key'] ??= 'default';
```

7. **箭头函数(PHP7.4+)**
```php
$fn = fn($x) => $x * 2;
```

8. **命名参数(PHP8.0+)**
```php
function named($a, $b) {}
named(b: 2, a: 1);
```

9. **联合类型(PHP8.0+)**
```php
function test(int|string $param): int|string {}
```

10. **match表达式(PHP8.0+)**
```php
$result = match($value) {
    1 => 'one',
    2 => 'two',
    default => 'other',
};
```