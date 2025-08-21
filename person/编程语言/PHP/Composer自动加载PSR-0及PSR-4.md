# Composer自动加载PSR-0及PSR-4

## PSR标准

关于PSR标准的详细介绍：<https://www.zybuluo.com/phper/note/65033>

> 目录结构说明
>
```md
|--composer.json
|--application
|     |--Controller
|          |--Test.php
|--vendor
|     |--autoload.php 
|--index.php
```

## PSR-0自动加载

1.在composer.json同级目录下新建文件：\application\Controller\Test.php，内容如下：

```php
<?php
namespace App\Controller;
class Test
{
    public function index()
    {
        echo "Test::index\r\n";
    }
}
```

2.修改composer.json文件，内容如下：
```json
"autoload":{
        "psr-0": {
            "App\\": "application/"
        }
}
```

3.修改index.php，内容如下：

```php
<?php
require 'vendor/autoload.php';
$c = new \App\Controller\Test();
$c->index();
```

4.更新composer配置文件


运行命令：`composer dump-autoload`

5.执行index.php：`php index.php`，报错了，提示找不到：`App\Test\index`类，原因为PSR-0会在application文件夹下去找`\App\Controller\Test.php`文件是否存在，存在则引入。但显然`\application\App\Controller\Test.php`文件并不存在。

解决办法：在application文件夹下新建一个App文件夹，然后把Test文件夹放进去就可以了


## PSR-4自动加载

PSR-4就简洁多了。

1.修改composer.json文件：

```json
"autoload":{
    "psr-4": {
        "App\\": "application/"
    }
}
```

2.更新composer配置文件
运行命令：`composer dump-autoload`

3.执行index.php `php index.php`
PSR-4会根据此路径：`\application\Controller\Test.php`找文件，存在则引入。跟PSR-0相比，就省略了App\。

## 自动加载流程分析

### psr-0

当composer.json中指定的是PSR-0加载方式时，执行`composer dump-autoload`命令后，产生了如下变化：

1.autoload_namespaces.php
该文件返回的数组中，会增加一个元素：

```php
return array(
    'App\\' => array($baseDir . '/application'),
);
```

内容为App对应的磁盘路径。

2.autoload_static.php
该文件新增了两个静态成员属性，`$prefixLengthsPsr4`和`$prefixDirsPsr4`。

```php
public static $prefixLengthsPsr4 = array (
    'A' => 
    array (
        'App\\' => 4,
    ),
);
public static $prefixDirsPsr4 = array (
    'App\\' => 
    array (
        0 => __DIR__ . '/../..' . '/application',
    ),
);
```

`$prefixLengthsPsr4`用类全名的第一个字母来做分类，方便后续的快速搜索。
`$prefixDirsPsr4`为类所在的磁盘路径。

## 自动加载过程

当程序执行到需要加载类文件时，首先调用的是`ClassLoader.php`文件的`loadClass`方法。

```php
public function loadClass($class)
{
    if ($file = $this->findFile($class)) {
        includeFile($file);
        return true;
    }
}
```

该方法中通过`findFile`方法来查找类文件，如果找到了，就通过`includeFile`引入进来。`findFile`方法的内容如下：

```php
public function findFile($class)
{
    // class map lookup
    if (isset($this->classMap[$class])) {
        return $this->classMap[$class];
    }
    if ($this->classMapAuthoritative || isset($this->missingClasses[$class])) {
        return false;
    }
    if (null !== $this->apcuPrefix) {
        $file = apcu_fetch($this->apcuPrefix.$class, $hit);
        if ($hit) {
            return $file;
        }
    }
    $file = $this->findFileWithExtension($class, '.php');
    // Search for Hack files if we are running on HHVM
    if (false === $file && defined('HHVM_VERSION')) {
        $file = $this->findFileWithExtension($class, '.hh');
    }
    if (null !== $this->apcuPrefix) {
        apcu_add($this->apcuPrefix.$class, $file);
    }
    if (false === $file) {
        // Remember that this class does not exist.
        $this->missingClasses[$class] = true;
    }
    return $file;
}
```

该方法首先检查自身的classMap数组中是否包含有该类，如果有，直接返回。如果没有则执行到`$this->findFileWithExtension`方法中。
`findFileWithExtension`方法的代码如下：

```php
private function findFileWithExtension($class, $ext)
    {
        // PSR-4 lookup
        $logicalPathPsr4 = strtr($class, '\\', DIRECTORY_SEPARATOR) . $ext;
        $first = $class[0];
        if (isset($this->prefixLengthsPsr4[$first])) {
            $subPath = $class;
            while (false !== $lastPos = strrpos($subPath, '\\')) {
                $subPath = substr($subPath, 0, $lastPos);
                $search = $subPath . '\\';
                if (isset($this->prefixDirsPsr4[$search])) {
                    $pathEnd = DIRECTORY_SEPARATOR . substr($logicalPathPsr4, $lastPos + 1);
                    foreach ($this->prefixDirsPsr4[$search] as $dir) {
                        if (file_exists($file = $dir . $pathEnd)) {
                            return $file;
                        }
                    }
                }
            }
        }
        // PSR-4 fallback dirs
        foreach ($this->fallbackDirsPsr4 as $dir) {
            if (file_exists($file = $dir . DIRECTORY_SEPARATOR . $logicalPathPsr4)) {
                return $file;
            }
        }
        // PSR-0 lookup
        if (false !== $pos = strrpos($class, '\\')) {
            // namespaced class name
            $logicalPathPsr0 = substr($logicalPathPsr4, 0, $pos + 1)
                . strtr(substr($logicalPathPsr4, $pos + 1), '_', DIRECTORY_SEPARATOR);
        } else {
            // PEAR-like class name
            $logicalPathPsr0 = strtr($class, '_', DIRECTORY_SEPARATOR) . $ext;
        }
        if (isset($this->prefixesPsr0[$first])) {
            foreach ($this->prefixesPsr0[$first] as $prefix => $dirs) {
                if (0 === strpos($class, $prefix)) {
                    foreach ($dirs as $dir) {
                        if (file_exists($file = $dir . DIRECTORY_SEPARATOR . $logicalPathPsr0)) {
                            return $file;
                        }
                    }
                }
            }
        }
        // PSR-0 fallback dirs
        foreach ($this->fallbackDirsPsr0 as $dir) {
            if (file_exists($file = $dir . DIRECTORY_SEPARATOR . $logicalPathPsr0)) {
                return $file;
            }
        }
        // PSR-0 include paths.
        if ($this->useIncludePath && $file = stream_resolve_include_path($logicalPathPsr0)) {
            return $file;
        }
        return false;
    }
```

可以看到，首先是按照PSR-4的标准，按类首字母进行循环，如果匹配到，则进一步匹配循环，如果找到类文件，则引入。如果按照PSR-4标准没有找到，则继续按PSR-0标准循环查找，如果找到类文件，则引入。