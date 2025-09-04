# PHP 基础速记手册

## 变量定义与访问

```php
// 基本定义
$a = 10;
$b = "hello";
$c = true;

// 变量变量
$varName = "foo";
$$varName = "bar"; // 等价于 $foo = "bar"

// 常量定义
define("PI", 3.14);
const MAX_USERS = 100;

// 预定义变量
$_GET, $_POST, $_REQUEST, $_SERVER, $_SESSION, $_COOKIE, $_FILES, $_ENV

// 变量作用域
global $globalVar; // 全局变量
static $staticVar; // 静态变量
```

## 数据类型

```php
// 标量类型
bool, int, float, string

// 复合类型
array, object, callable, iterable

// 特殊类型
resource, null

// 类型检测
is_int(), is_string(), is_array(), is_object(), is_callable()

// 类型转换
(int), (string), (array), (object), (bool)
settype($var, "integer")
```

## 数组操作

```php
// 数组定义
$arr1 = array(1, 2, 3);
$arr2 = [1, 2, 3]; // 短语法
$assoc = ["a" => 1, "b" => 2];

// 数组操作
count($arr), empty($arr)
array_push($arr, 4), array_pop($arr)
array_shift($arr), array_unshift($arr, 0)
array_merge($arr1, $arr2)
array_keys($assoc), array_values($assoc)
in_array(2, $arr), array_key_exists("a", $assoc)

// 数组遍历
foreach ($arr as $value) {}
foreach ($assoc as $key => $value) {}
```

## 字符串操作

```php
// 字符串连接
$str = "Hello" . " " . "World";

// 常用函数
strlen($str), substr($str, 0, 5)
strpos($str, "World"), str_replace("World", "PHP", $str)
trim($str), strtoupper($str), strtolower($str)
explode(" ", $str), implode("-", ["a", "b", "c"])
htmlspecialchars($str), htmlentities($str)
sprintf("Value: %d", 10)
```

## 类与对象

```php
// 类定义
class Person {
    // 属性
    public $name;
    protected $age;
    private $id;
    
    // 常量
    const SPECIES = "Human";
    
    // 构造方法
    public function __construct($name, $age) {
        $this->name = $name;
        $this->age = $age;
    }
    
    // 方法
    public function greet() {
        return "Hello, I'm " . $this->name;
    }
    
    // 静态方法
    public static function create($name) {
        return new self($name, 0);
    }
    
    // 魔术方法
    public function __toString() {
        return $this->name;
    }
}

// 继承
class Student extends Person {
    public function __construct($name, $age, $studentId) {
        parent::__construct($name, $age);
        $this->studentId = $studentId;
    }
    
    // 方法重写
    public function greet() {
        return parent::greet() . " (Student)";
    }
}

// 接口
interface Logger {
    public function log($message);
}

// 特性
trait Loggable {
    public function log($message) {
        echo $message;
    }
}

// 对象操作
$p = new Person("Alice", 25);
$p->greet();
Person::create("Bob");
$p instanceof Person;
```

## 函数

```php
// 函数定义
function add($a, $b) {
    return $a + $b;
}

// 默认参数
function greet($name = "World") {
    return "Hello, $name!";
}

// 可变参数
function sum(...$numbers) {
    return array_sum($numbers);
}

// 类型声明
function addNumbers(int $a, int $b): int {
    return $a + $b;
}

// 匿名函数
$square = function($x) {
    return $x * $x;
};

// 闭包
function makeMultiplier($factor) {
    return function($x) use ($factor) {
        return $x * $factor;
    };
}

// 回调函数
array_map(function($x) { return $x * 2; }, [1, 2, 3]);
```

## 控制结构

```php
// if-elseif-else
if ($a > $b) {
    echo "a is greater";
} elseif ($a == $b) {
    echo "a equals b";
} else {
    echo "a is smaller";
}

// switch
switch ($var) {
    case 1:
        echo "One";
        break;
    case 2:
        echo "Two";
        break;
    default:
        echo "Other";
}

// for循环
for ($i = 0; $i < 10; $i++) {
    echo $i;
}

// foreach循环
foreach ($array as $value) {
    echo $value;
}

foreach ($assoc as $key => $value) {
    echo "$key: $value";
}

// while/do-while
while ($i < 10) {
    echo $i++;
}

do {
    echo $i++;
} while ($i < 10);

// 异常处理
try {
    // 代码
} catch (Exception $e) {
    echo $e->getMessage();
} finally {
    // 清理代码
}
```

## 常用函数

```php
// 数学函数
abs(), ceil(), floor(), round()
min(), max(), rand(), mt_rand()
sqrt(), pow(), log(), sin(), cos()

// 日期时间
date("Y-m-d H:i:s"), time()
strtotime("next Monday")
DateTime::createFromFormat("Y-m-d", "2023-01-01")

// 文件系统
file_exists(), is_file(), is_dir()
file_get_contents(), file_put_contents()
fopen(), fread(), fwrite(), fclose()
mkdir(), rmdir(), unlink()
scandir(), glob()

// JSON
json_encode(), json_decode()

// 数据库(PDO)
$pdo = new PDO("mysql:host=localhost;dbname=test", "user", "pass");
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([1]);
$result = $stmt->fetchAll(PDO::FETCH_ASSOC);

// 会话管理
session_start(), session_destroy()
setcookie(), $_COOKIE

// 输出缓冲
ob_start(), ob_get_contents(), ob_end_clean(), ob_end_flush()

// 其他
header("Location: newpage.php") // 重定向
filter_var($email, FILTER_VALIDATE_EMAIL) // 验证邮箱
password_hash("password", PASSWORD_DEFAULT) // 密码哈希
password_verify("password", $hash) // 验证哈希
```

## 魔术方法

```php
__construct(), __destruct()
__call(), __callStatic()
__get(), __set(), __isset(), __unset()
__sleep(), __wakeup()
__toString(), __invoke()
__clone(), __debugInfo()
```

## 命名空间与自动加载

```php
// 命名空间
namespace MyProject;

class MyClass {
    // ...
}

// 使用命名空间
use MyProject\MyClass;
use AnotherProject\AnotherClass as AC;

// 自动加载
spl_autoload_register(function ($class) {
    include 'classes/' . $class . '.php';
});
```