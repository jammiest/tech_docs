# Laravel摘要

## 生命周期

> 更多信息请查看[https://laravel.p2hp.com/cndocs/11.x/lifecycle](https://laravel.p2hp.com/cndocs/11.x/lifecycle)

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

// 是否开启维护模式
if (file_exists($maintenance = __DIR__.'/../storage/framework/maintenance.php')) {
    require $maintenance;
}

// composer自动加载机制，加载项目依赖
require __DIR__.'/../vendor/autoload.php';

// 启动引导Laravel实例
/** @var Application $app */
$app = require_once __DIR__.'/../bootstrap/app.php';

// 处理请求
$app->handleRequest(Request::capture());
```