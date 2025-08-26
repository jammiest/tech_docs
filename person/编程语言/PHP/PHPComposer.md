# PHP Composer 详解

Composer 是 PHP 的依赖管理工具，类似于 Node.js 的 npm 或 Python 的 pip。它允许你声明项目所依赖的库，并为你管理（安装/更新）它们。

## 基本概念

### 1. 主要组件

- **composer.json**: 项目依赖声明文件
- **composer.lock**: 精确版本锁定文件
- **vendor/**: 依赖安装目录
- **autoload.php**: 自动加载文件

### 2. 安装 Composer

```bash
# 全局安装
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# 移动 composer.phar 到全局可执行目录
mv composer.phar /usr/local/bin/composer
```

## 核心功能

### 1. 初始化项目

```bash
composer init
```

这会引导你创建 `composer.json` 文件。

### 2. 安装依赖

```bash
composer install
```

这会读取 `composer.json` 并安装所有依赖到 `vendor/` 目录，同时生成 `composer.lock` 文件。

### 3. 添加新依赖

```bash
composer require vendor/package
```

例如：
```bash
composer require monolog/monolog
```

### 4. 更新依赖

```bash
# 更新所有依赖
composer update

# 更新特定包
composer update vendor/package
```

## composer.json 结构

```json
{
    "name": "your-project/name",
    "description": "Your project description",
    "type": "project",
    "require": {
        "php": "^7.4 || ^8.0",
        "monolog/monolog": "^2.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^9.0"
    },
    "autoload": {
        "psr-4": {
            "YourNamespace\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "YourNamespace\\Tests\\": "tests/"
        }
    }
}
```

## 版本约束

Composer 使用语义化版本控制：

| 约束     | 示例         | 说明               |
| -------- | ------------ | ------------------ |
| 精确版本 | `1.2.3`      | 完全匹配 1.2.3     |
| 范围     | `>=1.0 <2.0` | 1.0及以上，2.0以下 |
| 通配符   | `1.0.*`      | 1.0.0到1.0.999     |
| 波浪符   | `~1.2.3`     | >=1.2.3 <1.3.0     |
| 插入符   | `^1.2.3`     | >=1.2.3 <2.0.0     |

## 自动加载

Composer 提供了 PSR-4 和 PSR-0 自动加载标准：

```json
"autoload": {
    "psr-4": {
        "Acme\\": "src/",
        "Vendor\\Namespace\\": "lib/"
    },
    "files": ["src/helpers.php"]
}
```

在代码中使用自动加载：

```php
require __DIR__ . '/vendor/autoload.php';
```

## 脚本和事件

Composer 允许你在特定事件时运行脚本：

```json
"scripts": {
    "post-install-cmd": [
        "MyVendor\\MyClass::postInstall"
    ],
    "post-update-cmd": "php artisan optimize"
}
```

## 私有仓库和认证

可以在 `auth.json` 中配置私有仓库认证：

```json
{
    "http-basic": {
        "example.org": {
            "username": "your-username",
            "password": "your-password"
        }
    }
}
```

## 性能优化

```bash
# 安装时跳过开发依赖
composer install --no-dev

# 优化自动加载器
composer dump-autoload --optimize

# 使用并行安装
composer install --prefer-dist --optimize-autoloader --no-dev -j4
```

## 常见命令速查表

| 命令                     | 描述               |
| ------------------------ | ------------------ |
| `composer install`       | 安装依赖           |
| `composer update`        | 更新依赖           |
| `composer require`       | 添加新依赖         |
| `composer remove`        | 移除依赖           |
| `composer show`          | 显示已安装包       |
| `composer search`        | 搜索包             |
| `composer validate`      | 验证 composer.json |
| `composer outdated`      | 检查过时的包       |
| `composer dump-autoload` | 重新生成自动加载器 |

## 高级用法

### 1. 平台要求

```json
{
    "config": {
        "platform": {
            "php": "7.4.0",
            "ext-redis": "5.0.0"
        }
    }
}
```

### 2. 自定义安装路径

```json
{
    "config": {
        "vendor-dir": "custom_vendor",
        "bin-dir": "bin"
    }
}
```

### 3. 仓库配置

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/your/repo"
        },
        {
            "type": "composer",
            "url": "https://packages.example.com"
        }
    ]
}
```

Composer 是 PHP 生态系统中不可或缺的工具，熟练掌握它可以大大提高开发效率和项目管理能力。