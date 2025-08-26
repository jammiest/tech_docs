# Laravel 消息队列详解

Laravel 提供了强大的队列系统，允许你将耗时的任务（如发送电子邮件或处理文件）推迟处理，从而提高应用程序的响应速度。

## 1. 队列系统基础

### 1.1 配置

队列配置位于 `config/queue.php`，支持多种驱动：

- `sync`: 同步执行（用于开发）
- `database`: 数据库驱动
- `redis`: Redis 驱动
- `sqs`: AWS SQS
- `beanstalkd`: Beanstalkd

### 1.2 环境配置

`.env` 文件中的相关设置：

```ini
QUEUE_CONNECTION=database
QUEUE_DRIVER=database  # Laravel <5.7
```

## 2. 创建任务

### 2.1 生成任务类

```bash
php artisan make:job ProcessPodcast
```

生成的类位于 `app/Jobs` 目录，实现了 `ShouldQueue` 接口。

### 2.2 任务类示例

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use App\Models\Podcast;
use Illuminate\Support\Facades\Mail;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $tries = 5; // 最大尝试次数
    public $timeout = 120; // 超时时间（秒）
    public $maxExceptions = 3; // 最大异常次数
    public $backoff = [60, 120, 300]; // 重试间隔（秒）
    public $deleteWhenMissingModels = true; // 模型不存在时删除任务

    protected $podcast;

    /**
     * 创建一个新的任务实例
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * 执行任务
     */
    public function handle()
    {
        // 处理播客...
        Mail::to('user@example.com')->send(new PodcastProcessed($this->podcast));
    }

    /**
     * 处理失败任务
     */
    public function failed(Throwable $exception)
    {
        // 发送失败通知...
    }
}
```

## 3. 分发任务

### 3.1 基本分发

```php
ProcessPodcast::dispatch($podcast);
```

### 3.2 延迟分发

```php
// 延迟5分钟执行
ProcessPodcast::dispatch($podcast)->delay(now()->addMinutes(5));

// 指定时间执行
ProcessPodcast::dispatch($podcast)->delay(Carbon::tomorrow());
```

### 3.3 任务链

```php
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch($podcast);
```

### 3.4 批处理

```php
$batch = Bus::batch([
    new ProcessPodcast(1),
    new ProcessPodcast(2),
    new ProcessPodcast(3),
])->then(function (Batch $batch) {
    // 所有任务成功完成...
})->catch(function (Batch $batch, Throwable $e) {
    // 批处理中首次任务失败...
})->finally(function (Batch $batch) {
    // 批处理完成...
})->dispatch();
```

## 4. 队列工作进程

### 4.1 启动工作进程

```bash
php artisan queue:work
```

常用选项：

```bash
# 指定队列连接
php artisan queue:work redis

# 指定队列名称
php artisan queue:work --queue=high,default

# 处理单个任务后退出（用于调试）
php artisan queue:work --once

# 限制内存使用
php artisan queue:work --memory=128

# 超时设置
php artisan queue:work --timeout=60

# 睡眠时间（无任务时）
php artisan queue:work --sleep=3
```

### 4.2 守护进程

生产环境推荐使用 Supervisor 管理队列进程：

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /path/to/artisan queue:work --sleep=3 --tries=3 --max-jobs=1000
autostart=true
autorestart=true
user=www-data
numprocs=8
redirect_stderr=true
stdout_logfile=/path/to/laravel-worker.log
stopwaitsecs=3600
```

## 5. 数据库驱动

### 5.1 创建数据库表

```bash
php artisan queue:table
php artisan migrate
```

### 5.2 失败任务表

```bash
php artisan queue:failed-table
php artisan migrate
```

## 6. Redis 驱动

### 6.1 配置

`.env` 文件：

```ini
QUEUE_CONNECTION=redis
REDIS_QUEUE=database
```

`config/database.php`:

```php
'redis' => [
    'client' => 'predis',
    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_DB', 0),
    ],
    'queue' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_QUEUE_DB', 1),
    ],
],
```

## 7. 任务中间件

### 7.1 创建中间件

```bash
php artisan make:middleware RateLimited
```

### 7.2 中间件示例

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Queue\Middleware\RateLimited as QueueRateLimited;

class RateLimited
{
    public function handle($job, Closure $next)
    {
        return (new QueueRateLimited('backups'))
            ->handle($job, function () use ($job, $next) {
                return $next($job);
            });
    }
}
```

### 7.3 使用中间件

```php
class ProcessPodcast implements ShouldQueue
{
    public function middleware()
    {
        return [new RateLimited];
    }
}
```

## 8. 任务事件

可以在 `AppServiceProvider` 的 `boot` 方法中监听队列事件：

```php
use Illuminate\Support\Facades\Queue;
use Illuminate\Queue\Events\JobProcessing;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobFailed;

Queue::before(function (JobProcessing $event) {
    // 任务处理前...
});

Queue::after(function (JobProcessed $event) {
    // 任务处理后...
});

Queue::failing(function (JobFailed $event) {
    // 任务失败...
});

Queue::looping(function () {
    // 工作进程循环时...
});
```

## 9. 失败任务处理

### 9.1 查看失败任务

```bash
php artisan queue:failed
```

### 9.2 重试失败任务

```bash
# 重试所有失败任务
php artisan queue:retry all

# 重试特定ID的任务
php artisan queue:retry 1 2 3
```

### 9.3 删除失败任务

```bash
php artisan queue:forget 5
```

### 9.4 清空失败任务

```bash
php artisan queue:flush
```

## 10. 性能优化

### 10.1 批量插入任务

```php
$jobs = collect($podcasts)->map(function ($podcast) {
    return new ProcessPodcast($podcast);
});

Bus::batch($jobs)->dispatch();
```

### 10.2 使用 `dispatchAfterResponse`

```php
ProcessPodcast::dispatchAfterResponse($podcast);
```

### 10.3 队列优先级

```php
ProcessPodcast::dispatch($podcast)->onQueue('high');
```

然后启动不同优先级的工作进程：

```bash
php artisan queue:work --queue=high,default
```

## 11. 监控队列

### 11.1 Horizon (Redis 驱动)

安装 Laravel Horizon：

```bash
composer require laravel/horizon
php artisan horizon:install
```

### 11.2 自定义监控

可以使用 Prometheus + Grafana 监控队列指标：

```php
use Prometheus\CollectorRegistry;
use Prometheus\Storage\Redis;

$registry = new CollectorRegistry(new Redis([
    'host' => env('REDIS_HOST'),
    'port' => env('REDIS_PORT'),
    'password' => env('REDIS_PASSWORD'),
]));

$registry->getOrRegisterGauge(
    'laravel_queue',
    'jobs_processed_total',
    'Total number of processed jobs',
    ['queue']
);
```

Laravel 的队列系统提供了灵活而强大的异步任务处理能力，合理使用可以显著提升应用程序的性能和用户体验。