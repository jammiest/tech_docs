# Laravel 框架入门指南

Laravel 是一个优雅的 PHP Web 应用框架，以其简洁的语法和强大的功能而闻名。以下是 Laravel 的全面入门指南。

## 1. 环境准备

### 1.1 系统要求

- PHP ≥ 8.0
- Composer
- 数据库 (MySQL/PostgreSQL/SQLite/SQL Server)
- 可选: Node.js (前端开发)

### 1.2 安装 Laravel

```bash
# 通过 Composer 创建项目
composer create-project laravel/laravel my-project

# 或使用 Laravel 安装器
composer global require laravel/installer
laravel new my-project
```

## 2. 项目结构

```
my-project/
├── app/                # 核心应用代码
│   ├── Console/        # Artisan 命令
│   ├── Exceptions/     # 异常处理
│   ├── Http/          # 控制器、中间件等
│   ├── Models/        # 数据模型
│   └── Providers/     # 服务提供者
├── bootstrap/          # 框架启动文件
├── config/            # 配置文件
├── database/          # 数据库迁移和种子
├── public/            # 公开访问目录
├── resources/         # 视图、语言文件等
├── routes/            # 路由定义
├── storage/           # 日志、缓存等
├── tests/             # 测试代码
└── vendor/            # Composer 依赖
```

## 3. 基本配置

### 3.1 环境配置

编辑 `.env` 文件：

```ini
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:...  # 自动生成
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

CACHE_DRIVER=file
QUEUE_CONNECTION=sync
```

生成应用密钥：

```bash
php artisan key:generate
```

## 4. 路由系统

### 4.1 基本路由

`routes/web.php`:

```php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/hello', function () {
    return 'Hello World';
});

Route::get('/user/{id}', function ($id) {
    return 'User '.$id;
});
```

### 4.2 路由参数

```php
Route::get('/post/{post}/comment/{comment}', function ($postId, $commentId) {
    return "Post $postId, Comment $commentId";
});

// 可选参数
Route::get('/user/{name?}', function ($name = 'Guest') {
    return "Hello $name";
});

// 正则约束
Route::get('/user/{id}', function ($id) {
    return "User $id";
})->where('id', '[0-9]+');
```

### 4.3 路由组

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // 匹配 "/admin/users" URL
    });
    
    Route::get('/posts', function () {
        // 匹配 "/admin/posts" URL
    });
});
```

## 5. 控制器

### 5.1 创建控制器

```bash
php artisan make:controller UserController
```

### 5.2 控制器示例

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index()
    {
        $users = User::all();
        return view('users.index', ['users' => $users]);
    }

    public function show($id)
    {
        $user = User::findOrFail($id);
        return view('users.show', ['user' => $user]);
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|max:255',
            'email' => 'required|email|unique:users',
        ]);
        
        $user = User::create($validated);
        return redirect('/users/'.$user->id);
    }
}
```

### 5.3 资源路由

```php
Route::resource('users', UserController::class);
```

这会自动创建以下路由：

| 方法 | URI | 动作 | 路由名称 |
|------|-----|------|----------|
| GET | /users | index | users.index |
| GET | /users/create | create | users.create |
| POST | /users | store | users.store |
| GET | /users/{user} | show | users.show |
| GET | /users/{user}/edit | edit | users.edit |
| PUT/PATCH | /users/{user} | update | users.update |
| DELETE | /users/{user} | destroy | users.destroy |

## 6. 数据库与Eloquent ORM

### 6.1 创建模型和迁移

```bash
php artisan make:model Post -m
```

这会创建 `app/Models/Post.php` 和 `database/migrations/..._create_posts_table.php`

### 6.2 迁移文件示例

```php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('content');
        $table->foreignId('user_id')->constrained();
        $table->timestamps();
    });
}
```

运行迁移：

```bash
php artisan migrate
```

### 6.3 Eloquent 模型

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected $fillable = ['title', 'content', 'user_id'];
    
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

### 6.4 基本查询

```php
// 获取所有记录
$posts = Post::all();

// 条件查询
$posts = Post::where('active', 1)
            ->orderBy('created_at', 'desc')
            ->take(10)
            ->get();

// 查找单个记录
$post = Post::find(1);

// 创建记录
$post = Post::create([
    'title' => 'My first post',
    'content' => 'Content here'
]);

// 更新记录
$post->title = 'New title';
$post->save();

// 删除记录
$post->delete();
```

## 7. 视图与Blade模板

### 7.1 创建视图

`resources/views/welcome.blade.php`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Hello, {{ $name }}</h1>
    
    @if ($items)
        <ul>
            @foreach ($items as $item)
                <li>{{ $item }}</li>
            @endforeach
        </ul>
    @else
        <p>No items found.</p>
    @endif
</body>
</html>
```

### 7.2 渲染视图

```php
Route::get('/', function () {
    return view('welcome', [
        'name' => 'John',
        'items' => ['Apple', 'Banana', 'Orange']
    ]);
});
```

### 7.3 Blade 常用指令

```html
{{-- 注释 --}}

{{ $variable }}           {{-- 转义输出 --}}
{!! $html !!}             {{-- 原始输出 --}}

@if (count($items) > 0)   {{-- 条件语句 --}}
    <p>Items found</p>
@else
    <p>No items</p>
@endif

@foreach ($users as $user) {{-- 循环 --}}
    <p>{{ $user->name }}</p>
@endforeach

@include('partials.header') {{-- 包含子视图 --}}

@extends('layouts.app')    {{-- 继承布局 --}}
@section('content')       {{-- 定义区块 --}}
    <p>Main content</p>
@endsection
```

## 8. 表单处理

### 8.1 创建表单

```html
<form method="POST" action="/posts">
    @csrf
    
    <div>
        <label for="title">Title</label>
        <input type="text" id="title" name="title" value="{{ old('title') }}">
        @error('title')
            <div class="error">{{ $message }}</div>
        @enderror
    </div>
    
    <div>
        <label for="content">Content</label>
        <textarea id="content" name="content">{{ old('content') }}</textarea>
        @error('content')
            <div class="error">{{ $message }}</div>
        @enderror
    </div>
    
    <button type="submit">Create Post</button>
</form>
```

### 8.2 处理表单

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'content' => 'required',
    ]);
    
    $post = Post::create($validated);
    
    return redirect('/posts/'.$post->id);
}
```

## 9. 用户认证

### 9.1 安装认证脚手架

```bash
composer require laravel/ui
php artisan ui bootstrap --auth
npm install && npm run dev
```

### 9.2 用户模型

Laravel 自带 `App\Models\User` 模型，位于 `app/Models/User.php`

### 9.3 路由保护

```php
Route::get('/profile', function () {
    // 只有认证用户可以访问
})->middleware('auth');
```

### 9.4 手动认证

```php
use Illuminate\Support\Facades\Auth;

// 登录用户
if (Auth::attempt(['email' => $email, 'password' => $password])) {
    // 认证通过...
    return redirect()->intended('dashboard');
}

// 获取当前用户
$user = Auth::user();

// 登出
Auth::logout();
```

## 10. 测试与部署

### 10.1 运行测试

```bash
php artisan test
```

### 10.2 部署准备

```bash
# 优化自动加载器
composer install --optimize-autoloader --no-dev

# 缓存配置和路由
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## 11. 常用 Artisan 命令

| 命令 | 描述 |
|------|------|
| `php artisan serve` | 启动开发服务器 |
| `php artisan make:model` | 创建模型 |
| `php artisan make:controller` | 创建控制器 |
| `php artisan make:migration` | 创建迁移 |
| `php artisan migrate` | 运行迁移 |
| `php artisan db:seed` | 运行数据填充 |
| `php artisan tinker` | 交互式 REPL |
| `php artisan storage:link` | 创建存储链接 |
| `php artisan optimize` | 优化应用性能 |
| `php artisan queue:work` | 启动队列工作进程 |

Laravel 提供了丰富的功能和优雅的语法，使 PHP 开发变得更加愉快和高效。通过掌握这些基础知识，你已经可以开始构建功能完善的 Web 应用了。