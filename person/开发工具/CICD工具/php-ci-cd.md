# PHP 项目 CI/CD

## 1. 概述

PHP 项目的 CI/CD 流程需要针对 PHP 生态系统的特点进行优化，包括依赖管理、测试框架、代码质量工具和部署策略。本指南提供完整的 PHP 项目 CI/CD 实施方案，涵盖从代码提交到生产部署的全流程。

## 2. 环境配置

### 2.1 基础环境设置
```bash
#!/bin/bash
# setup-php-environment.sh

# 安装 PHP 和扩展
sudo apt update
sudo apt install -y php8.1 php8.1-cli php8.1-fpm \
php8.1-mysql php8.1-pgsql php8.1-sqlite3 \
php8.1-curl php8.1-gd php8.1-mbstring \
php8.1-xml php8.1-zip php8.1-intl \
php8.1-redis php8.1-memcached

# 安装 Composer
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer

# 配置 PHP
sudo sed -i 's/memory_limit = .*/memory_limit = 512M/' /etc/php/8.1/cli/php.ini
sudo sed -i 's/upload_max_filesize = .*/upload_max_filesize = 50M/' /etc/php/8.1/fpm/php.ini
sudo sed -i 's/post_max_size = .*/post_max_size = 50M/' /etc/php/8.1/fpm/php.ini

# 安装测试工具
composer global require phpunit/phpunit
composer global require friendsofphp/php-cs-fixer
composer global require phpstan/phpstan

# 验证安装
php -v
composer --version
phpunit --version
```

### 2.2 Docker 化 PHP 环境
```dockerfile
# Dockerfile
FROM php:8.1-fpm

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    libzip-dev

# 安装 PHP 扩展
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip

# 安装 Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 设置工作目录
WORKDIR /var/www

# 复制应用文件
COPY . .

# 安装依赖
RUN composer install --no-dev --optimize-autoloader

# 设置权限
RUN chown -R www-data:www-data /var/www

# 暴露端口
EXPOSE 9000

CMD ["php-fpm"]
```

## 3. CI/CD 流水线设计

### 3.1 GitHub Actions 配置
```yaml
# .github/workflows/php-ci-cd.yml
name: PHP CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  PHP_VERSION: '8.1'
  COMPOSER_FLAGS: '--prefer-dist --no-interaction'

jobs:
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ env.PHP_VERSION }}
          extensions: mbstring, xml, curl, gd, zip
          coverage: xdebug

      - name: Install dependencies
        run: composer install ${{ env.COMPOSER_FLAGS }}

      - name: Run PHPStan
        run: vendor/bin/phpstan analyse --level=8

      - name: Run PHP CS Fixer
        run: vendor/bin/php-cs-fixer fix --dry-run --diff

      - name: Run security check
        run: composer require --dev roave/security-advisories:dev-latest && composer update

  tests:
    runs-on: ubuntu-latest
    needs: code-quality
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: test
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ env.PHP_VERSION }}
          extensions: mbstring, xml, curl, gd, zip
          coverage: xdebug

      - name: Install dependencies
        run: composer install ${{ env.COMPOSER_FLAGS }}

      - name: Copy environment file
        run: cp .env.testing .env

      - name: Generate application key
        run: php artisan key:generate

      - name: Run migrations
        run: php artisan migrate --force

      - name: Run tests
        run: vendor/bin/phpunit --coverage-clover coverage.xml

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: coverage.xml

  deploy:
    runs-on: ubuntu-latest
    needs: tests
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ env.PHP_VERSION }}

      - name: Install dependencies
        run: composer install --no-dev --optimize-autoloader

      - name: Deploy to production
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.PRODUCTION_HOST }}
          username: ${{ secrets.PRODUCTION_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/production
            git pull origin main
            composer install --no-dev --optimize-autoloader
            php artisan migrate --force
            php artisan optimize:clear
            php artisan optimize
            sudo systemctl reload php8.1-fpm
```

### 3.2 GitLab CI 配置
```yaml
# .gitlab-ci.yml
image: php:8.1

stages:
  - code_quality
  - test
  - deploy

variables:
  COMPOSER_FLAGS: "--prefer-dist --no-interaction"

before_script:
  - apt-get update && apt-get install -y git unzip
  - curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
  - composer install $COMPOSER_FLAGS

code_quality:
  stage: code_quality
  script:
    - vendor/bin/phpstan analyse --level=8
    - vendor/bin/php-cs-fixer fix --dry-run --diff
    - composer require --dev roave/security-advisories:dev-latest && composer update

test:
  stage: test
  services:
    - mysql:8.0
  variables:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: test
  script:
    - cp .env.testing .env
    - php artisan key:generate
    - php artisan migrate --force
    - vendor/bin/phpunit --coverage-text

deploy_production:
  stage: deploy
  environment: production
  only:
    - main
  script:
    - apt-get install -y openssh-client
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa
    - ssh-keyscan -H $PRODUCTION_HOST >> ~/.ssh/known_hosts
    - ssh $PRODUCTION_USER@$PRODUCTION_HOST "cd /var/www/production && git pull && composer install --no-dev --optimize-autoloader && php artisan migrate --force && php artisan optimize"
```

## 4. 代码质量工具

### 4.1 PHPStan 配置
```neon
# phpstan.neon
parameters:
  level: 8
  paths:
    - app
    - src
  excludePaths:
    - tests
    - vendor
    - storage
  ignoreErrors:
    - '#Call to method .* on an unknown class#'
  reportUnmatchedIgnoredErrors: false
  checkMissingIterableValueType: false

includes:
  - vendor/phpstan/phpstan-deprecation-rules/rules.neon
  - vendor/phpstan/phpstan-strict-rules/rules.neon
```

### 4.2 PHP CS Fixer 配置
```php
# .php-cs-fixer.php
<?php

$finder = PhpCsFixer\Finder::create()
    ->in(__DIR__)
    ->exclude('vendor')
    ->exclude('storage')
    ->exclude('bootstrap/cache')
    ->name('*.php')
    ->notName('*.blade.php')
    ->ignoreDotFiles(true)
    ->ignoreVCS(true);

return PhpCsFixer\Config::create()
    ->setRules([
        '@PSR12' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sortAlgorithm' => 'alpha'],
        'no_unused_imports' => true,
        'not_operator_with_successor_space' => true,
        'trailing_comma_in_multiline' => true,
        'phpdoc_scalar' => true,
        'unary_operator_spaces' => true,
        'binary_operator_spaces' => true,
        'blank_line_before_statement' => [
            'statements' => ['break', 'continue', 'declare', 'return', 'throw', 'try'],
        ],
        'phpdoc_single_line_var_spacing' => true,
        'phpdoc_var_without_name' => true,
        'method_argument_space' => [
            'on_multiline' => 'ensure_fully_multiline',
            'keep_multiple_spaces_after_comma' => true,
        ],
    ])
    ->setFinder($finder);
```

## 5. 测试策略

### 5.1 PHPUnit 配置
```xml
<!-- phpunit.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/9.3/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         verbose="true">
    <testsuites>
        <testsuite name="Unit">
            <directory suffix="Test.php">./tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory suffix="Test.php">./tests/Feature</directory>
        </testsuite>
    </testsuites>
    <coverage processUncoveredFiles="true">
        <include>
            <directory suffix=".php">./app</directory>
            <directory suffix=".php">./src</directory>
        </include>
        <exclude>
            <directory>./vendor</directory>
            <directory>./storage</directory>
            <directory>./bootstrap/cache</directory>
        </exclude>
    </coverage>
    <php>
        <server name="APP_ENV" value="testing"/>
        <server name="BCRYPT_ROUNDS" value="4"/>
        <server name="CACHE_DRIVER" value="array"/>
        <server name="DB_CONNECTION" value="sqlite"/>
        <server name="DB_DATABASE" value=":memory:"/>
        <server name="MAIL_MAILER" value="array"/>
        <server name="QUEUE_CONNECTION" value="sync"/>
        <server name="SESSION_DRIVER" value="array"/>
        <server name="TELESCOPE_ENABLED" value="false"/>
    </php>
</phpunit>
```

### 5.2 数据库测试配置
```php
<?php
// tests/TestCase.php

namespace Tests;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use DatabaseMigrations, DatabaseTransactions;

    protected function setUp(): void
    {
        parent::setUp();
        $this->withoutExceptionHandling();
    }

    protected function createTestData()
    {
        // 通用的测试数据创建逻辑
    }

    protected function assertDatabaseCount($table, $count)
    {
        $this->assertEquals($count, \DB::table($table)->count());
    }
}
```

## 6. 部署策略

### 6.1 零停机部署
```bash
#!/bin/bash
# deploy-with-zero-downtime.sh

set -e

echo "开始零停机部署..."

# 切换到部署目录
cd /var/www/production

# 拉取最新代码
git fetch origin
git checkout main
git pull origin main

# 安装依赖
composer install --no-dev --optimize-autoloader --no-interaction

# 运行数据库迁移
php artisan migrate --force

# 清除缓存
php artisan optimize:clear

# 热重载 PHP-FPM
sudo systemctl reload php8.1-fpm

# 健康检查
curl -f http://localhost/health > /dev/null 2>&1

echo "部署完成！"
```

### 6.2 回滚脚本
```bash
#!/bin/bash
# rollback-deployment.sh

set -e

echo "开始回滚部署..."

# 切换到部署目录
cd /var/www/production

# 回退到上一个提交
git reset --hard HEAD@{1}

# 安装依赖
composer install --no-dev --optimize-autoloader --no-interaction

# 回滚数据库迁移
php artisan migrate:rollback --step=1 --force

# 清除缓存
php artisan optimize:clear

# 热重载 PHP-FPM
sudo systemctl reload php8.1-fpm

echo "回滚完成！"
```

## 7. 监控与日志

### 7.1 应用健康检查
```php
<?php
// routes/health.php

Route::get('/health', function () {
    try {
        // 检查数据库连接
        DB::connection()->getPdo();
        
        // 检查 Redis 连接
        Redis::connection()->ping();
        
        // 检查存储权限
        Storage::disk('local')->put('healthcheck', 'ok');
        Storage::disk('local')->delete('healthcheck');
        
        return response()->json([
            'status' => 'healthy',
            'timestamp' => now(),
            'services' => [
                'database' => 'connected',
                'redis' => 'connected',
                'storage' => 'writable'
            ]
        ]);
    } catch (\Exception $e) {
        Log::error('Health check failed: ' . $e->getMessage());
        
        return response()->json([
            'status' => 'unhealthy',
            'error' => $e->getMessage(),
            'timestamp' => now()
        ], 500);
    }
});
```

### 7.2 性能监控
```yaml
# config/monitoring.php
<?php

return [
    'metrics' => [
        'enabled' => env('METRICS_ENABLED', true),
        'driver' => env('METRICS_DRIVER', 'prometheus'),
        
        'collectors' => [
            'response_time' => [
                'buckets' => [0.1, 0.5, 1, 2, 5]
            ],
            'memory_usage' => [
                'buckets' => [10, 50, 100, 200, 500]
            ],
            'database_queries' => [
                'buckets' => [10, 50, 100, 200, 500]
            ]
        ]
    ],
    
    'alerts' => [
        'response_time' => env('ALERT_RESPONSE_TIME', 1000),
        'memory_usage' => env('ALERT_MEMORY_USAGE', 128),
        'error_rate' => env('ALERT_ERROR_RATE', 0.01)
    ]
];
```

## 8. 安全最佳实践

### 8.1 安全扫描集成
```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'  # 每周日运行
  push:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.1'

      - name: Install dependencies
        run: composer install

      - name: Run security audit
        run: composer audit

      - name: Run PHPStan security analysis
        run: vendor/bin/phpstan analyse --level=max --configuration=phpstan-security.neon

      - name: Run dependency vulnerability scan
        uses: dependency-check/DependencyCheck@main
        with:
          project: 'My PHP Project'
          path: '.'
          format: 'HTML'
          out: 'reports'

      - name: Upload security report
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: reports
```

### 8.2 环境安全配置
```bash
#!/bin/bash
# secure-environment.sh

# 设置文件权限
find /var/www -type f -exec chmod 644 {} \;
find /var/www -type d -exec chmod 755 {} \;
chmod -R 775 /var/www/storage
chmod -R 775 /var/www/bootstrap/cache

# 保护环境文件
chmod 600 /var/www/.env
chown www-data:www-data /var/www/.env

# 配置 PHP 安全设置
sed -i 's/expose_php = On/expose_php = Off/' /etc/php/8.1/fpm/php.ini
sed -i 's/display_errors = On/display_errors = Off/' /etc/php/8.1/fpm/php.ini
sed -i 's/allow_url_include = On/allow_url_include = Off/' /etc/php/8.1/fpm/php.ini

# 重启 PHP-FPM
systemctl reload php8.1-fpm

echo "环境安全配置完成"
```
