# ThinkPHP 框架入门指南

ThinkPHP 是一个快速、简单的面向对象的轻量级 PHP 开发框架，特别适合国内的中小型项目开发。以下是 ThinkPHP 6.x 版本的全面入门指南。

## 1. 环境准备与安装

### 1.1 系统要求

- PHP ≥ 7.1 (推荐 PHP 7.4+)
- Composer
- PDO PHP 扩展
- MBstring PHP 扩展

### 1.2 安装 ThinkPHP

```bash
# 创建项目
composer create-project topthink/think tp6

# 进入项目目录
cd tp6
```

### 1.3 目录结构

```
tp6/
├── app/                # 应用目录
│   ├── controller/     # 控制器
│   ├── model/          # 模型
│   └── view/           # 视图
├── config/             # 配置文件
├── public/             # 入口文件
├── route/              # 路由文件
├── runtime/            # 运行时目录
├── vendor/             # Composer 依赖
└── extend/             # 扩展目录
```

## 2. 基本配置

### 2.1 应用配置

`config/app.php`:

```php
return [
    'app_name' => 'My Application',
    'app_debug' => true,
    'default_timezone' => 'Asia/Shanghai',
];
```

### 2.2 数据库配置

`config/database.php`:

```php
return [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [
            'type' => 'mysql',
            'hostname' => '127.0.0.1',
            'database' => 'tp6',
            'username' => 'root',
            'password' => '',
            'charset' => 'utf8mb4',
            'prefix' => 'tp_',
        ],
    ],
];
```

## 3. 路由系统

### 3.1 基本路由

`route/app.php`:

```php
use think\facade\Route;

// 基本GET路由
Route::get('hello', function () {
    return 'Hello, ThinkPHP!';
});

// 带参数路由
Route::get('user/:id', 'User/read');

// 路由分组
Route::group('blog', function () {
    Route::get(':id', 'Blog/read');
    Route::post('save', 'Blog/save');
})->ext('html');
```

### 3.2 RESTful 路由

```php
Route::resource('blog', 'Blog');
```

这会自动创建以下路由：

| 方法 | URI | 操作 | 说明 |
|------|-----|------|------|
| GET | blog | index | 显示博客列表 |
| GET | blog/create | create | 显示创建表单 |
| POST | blog | save | 保存博客 |
| GET | blog/:id | read | 显示单篇博客 |
| GET | blog/:id/edit | edit | 显示编辑表单 |
| PUT | blog/:id | update | 更新博客 |
| DELETE | blog/:id | delete | 删除博客 |

## 4. 控制器

### 4.1 创建控制器

```bash
php think make:controller Blog
```

生成 `app/controller/Blog.php`:

```php
<?php

namespace app\controller;

use think\Request;

class Blog
{
    public function index()
    {
        return '博客列表';
    }

    public function read($id)
    {
        return '查看博客ID: ' . $id;
    }

    public function save(Request $request)
    {
        return '保存博客: ' . $request->param('title');
    }
}
```

### 4.2 请求与响应

```php
use think\facade\Request;
use think\Response;

public function read($id)
{
    // 获取请求参数
    $name = Request::param('name');
    
    // JSON响应
    return json([
        'id' => $id,
        'name' => $name
    ]);
    
    // 视图响应
    return view('index', ['name' => $name]);
    
    // 重定向
    return redirect('/hello');
}
```

## 5. 数据库与模型

### 5.1 创建模型

```bash
php think make:model Blog
```

生成 `app/model/Blog.php`:

```php
<?php

namespace app\model;

use think\Model;

class Blog extends Model
{
    // 设置表名 (默认为小写类名)
    protected $table = 'blog';
    
    // 定义时间戳字段
    protected $createTime = 'create_time';
    protected $updateTime = 'update_time';
    
    // 关联用户
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

### 5.2 基本查询

```php
// 查询所有记录
$list = Blog::select();

// 条件查询
$list = Blog::where('status', 1)
           ->order('create_time', 'desc')
           ->limit(10)
           ->select();

// 查找单条记录
$blog = Blog::find($id);

// 创建记录
$blog = Blog::create([
    'title' => 'ThinkPHP入门',
    'content' => '内容...'
]);

// 更新记录
$blog->title = '新标题';
$blog->save();

// 删除记录
$blog->delete();
```

## 6. 视图与模板

### 6.1 创建视图

`app/view/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{$title}</title>
</head>
<body>
    <h1>{$title}</h1>
    
    {if condition="$list"}
        <ul>
            {volist name="list" id="vo"}
                <li>{$vo.title} - {$vo.create_time|date='Y-m-d'}</li>
            {/volist}
        </ul>
    {else /}
        <p>暂无数据</p>
    {/if}
</body>
</html>
```

### 6.2 渲染视图

```php
public function index()
{
    $list = Blog::select();
    return view('index', [
        'title' => '博客列表',
        'list' => $list
    ]);
}
```

### 6.3 模板常用标签

```html
<!-- 变量输出 -->
{$name}
{$blog.title}

<!-- 条件判断 -->
{if condition="$score >= 60"}
    及格
{elseif condition="$score >= 50" /}
    补考
{else /}
    不及格
{/if}

<!-- 循环 -->
{volist name="list" id="vo" key="k"}
    {$k}. {$vo.name}
{/volist}

<!-- 包含子模板 -->
{include file="public/header" /}

<!-- 原生PHP -->
{php}echo date('Y-m-d');{/php}
```

## 7. 表单处理

### 7.1 创建表单

```html
<form action="{:url('blog/save')}" method="post">
    <input type="text" name="title" placeholder="标题">
    <textarea name="content" placeholder="内容"></textarea>
    <button type="submit">提交</button>
</form>
```

### 7.2 表单验证

```php
use think\facade\Validate;

public function save(Request $request)
{
    $data = $request->post();
    
    $validate = Validate::rule([
        'title|标题' => 'require|max:100',
        'content|内容' => 'require',
    ]);
    
    if (!$validate->check($data)) {
        return $validate->getError();
    }
    
    $blog = Blog::create($data);
    return '保存成功: ' . $blog->id;
}
```

## 8. 中间件

### 8.1 创建中间件

```bash
php think make:middleware Auth
```

生成 `app/middleware/Auth.php`:

```php
<?php

namespace app\middleware;

class Auth
{
    public function handle($request, \Closure $next)
    {
        if (!$request->session('user')) {
            return redirect('/login');
        }
        
        return $next($request);
    }
}
```

### 8.2 使用中间件

```php
// 路由中使用
Route::rule('profile', 'User/profile')->middleware(Auth::class);

// 控制器中使用
protected $middleware = [
    Auth::class => ['except' => ['login', 'register']]
];
```

## 9. 会话与Cookie

### 9.1 会话操作

```php
// 设置session
session('user', $user);

// 获取session
$user = session('user');

// 删除session
session('user', null);

// 清空session
session(null);
```

### 9.2 Cookie操作

```php
// 设置cookie
cookie('name', 'value', 3600);

// 获取cookie
$value = cookie('name');

// 删除cookie
cookie('name', null);
```

## 10. 文件上传

```php
public function upload(Request $request)
{
    $file = $request->file('image');
    
    if ($file) {
        $savePath = 'uploads/' . date('Ymd');
        $info = $file->validate(['size'=>1024000,'ext'=>'jpg,png,gif'])
                    ->move($savePath);
        
        if ($info) {
            return '上传成功: ' . $info->getSaveName();
        } else {
            return $file->getError();
        }
    }
    
    return '请选择文件';
}
```

## 11. 常用命令

| 命令 | 描述 |
|------|------|
| `php think run` | 启动开发服务器 |
| `php think make:controller` | 创建控制器 |
| `php think make:model` | 创建模型 |
| `php think make:middleware` | 创建中间件 |
| `php think migrate:run` | 运行数据库迁移 |
| `php think optimize:schema` | 生成数据表字段缓存 |
| `php think clear` | 清除缓存文件 |
| `php think version` | 查看框架版本 |

## 12. 部署优化

```bash
# 生成路由缓存
php think optimize:route

# 生成配置缓存
php think optimize:config

# 预加载框架文件
php think optimize:schema

# 生产环境关闭调试模式
修改 .env 文件：
APP_DEBUG = false
```

ThinkPHP 6.x 采用了更加现代化的架构设计，同时保持了简单易用的特点。通过掌握这些基础知识，你已经可以开始使用 ThinkPHP 开发 Web 应用了。