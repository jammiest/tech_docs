
# **一、安装与配置**
## 1. 安装 Composer
```bash
# Linux/macOS
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer

# Windows
下载并运行 https://getcomposer.org/Composer-Setup.exe
```

## 2. 配置全局镜像（中国用户）
```bash
# 阿里云镜像
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# 腾讯云镜像
composer config -g repo.packagist composer https://mirrors.cloud.tencent.com/composer/

# 恢复默认
composer config -g --unset repos.packagist
```

## 3. 全局配置
```bash
# 查看所有配置
composer config -l

# 修改全局安装路径
composer config -g home /path/to/composer_home

# 设置代理
composer config -g http-proxy http://127.0.0.1:1080
```

---

# **二、核心命令速查**
## 1. 项目初始化与管理
| 命令 | 说明 |
|------|------|
| `composer init` | 交互式创建 `composer.json` |
| `composer install` | 安装 `composer.lock` 中的依赖 |
| `composer update` | 更新所有依赖到最新版本 |
| `composer update vendor/package` | 更新指定包 |
| `composer require vendor/package` | 安装新包 |
| `composer remove vendor/package` | 移除包 |
| `composer dump-autoload` | 重新生成自动加载文件 |

## 2. 包查询与检查
| 命令 | 说明 |
|------|------|
| `composer show` | 列出所有已安装包 |
| `composer show vendor/package` | 查看包详情 |
| `composer search 关键词` | 搜索包 |
| `composer outdated` | 检查过时的包 |
| `composer validate` | 验证 `composer.json` |

## 3. 全局包管理
```bash
# 安装全局工具（如 Laravel 安装器）
composer global require laravel/installer

# 全局包列表
composer global show

# 更新全局包
composer global update
```

---

# **三、版本控制语法**
| 语法 | 示例 | 说明 |
|------|------|------|
| `^1.2.3` | `^8.0` | 允许 1.2.3 ≤ 版本 < 2.0.0 |
| `~1.2.3` | `~8.0.0` | 允许 1.2.3 ≤ 版本 < 1.3.0 |
| `1.2.*` | `8.*` | 允许 1.2.0 ≤ 版本 < 1.3.0 |
| `dev-master` | `dev-main` | 使用分支最新代码 |
| `1.2.3 - 2.0.0` | `8.0 - 9.0` | 版本范围 |
| `@stable` | `@dev` | 稳定性标记 |

---

# **四、`composer.json` 全配置**
```json
{
  "name": "vendor/project",
  "description": "项目描述",
  "type": "project",
  "license": "MIT",
  "require": {
    "php": "^8.0",
    "laravel/framework": "^9.0"
  },
  "require-dev": {
    "phpunit/phpunit": "^9.0"
  },
  "autoload": {
    "psr-4": {
      "App\\": "app/",
      "Database\\Factories\\": "database/factories/"
    },
    "files": ["helpers.php"]
  },
  "autoload-dev": {
    "psr-4": {
      "Tests\\": "tests/"
    }
  },
  "scripts": {
    "post-autoload-dump": [
      "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
      "@php artisan package:discover"
    ],
    "post-update-cmd": [
      "@php artisan vendor:publish --tag=laravel-assets"
    ]
  },
  "minimum-stability": "dev",
  "prefer-stable": true,
  "config": {
    "optimize-autoloader": true,
    "preferred-install": "dist",
    "sort-packages": true
  },
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/your/private-repo"
    }
  ]
}
```

---

# **五、高级用法**
## 1. 加速安装
```bash
# 使用并行安装（Composer 2.1+）
composer install --prefer-dist --optimize-autoloader --apcu-autoloader

# 跳过开发依赖
composer install --no-dev
```

## 2. 私有仓库配置
```bash
# 添加私有仓库
composer config repositories.foo vcs https://github.com/your/private-repo

# 认证配置
composer config http-basic.example.com username password
```

## 3. 脚本钩子示例
```json
"scripts": {
  "test": "@php vendor/bin/phpunit",
  "deploy": [
    "@composer install --no-dev",
    "@php artisan migrate --force",
    "@php artisan cache:clear"
  ]
}
```

---

# **六、问题排查大全**
## 1. 内存不足
```bash
COMPOSER_MEMORY_LIMIT=-1 composer update
```

## 2. 版本冲突
```bash
# 查看冲突原因
composer why vendor/package

# 强制更新依赖树
composer update --with-all-dependencies
```

## 3. 常见错误
| 错误 | 解决方案 |
|------|------|
| `Could not find package` | 检查镜像源/包名拼写/版本约束 |
| `Your requirements could not be resolved` | 使用 `composer update -vvv` 查看详情 |
| `Class not found` | 运行 `composer dump-autoload` |
| `The requested package exists but is not provable` | 添加 `--with-all-dependencies` |

## 4. 调试模式
```bash
# 显示详细日志
composer -vvv install

# 诊断环境
composer diagnose
```

---

# **七、工作流示例**
## 1. 创建新项目
```bash
composer create-project laravel/laravel my-project
```

## 2. 开发流程
```bash
# 安装开发依赖
composer require --dev phpstan/phpstan

# 本地开发时加载类
composer dump-autoload

# 准备生产环境
composer install --optimize-autoloader --no-dev
```

## 3. 发布包
```bash
# 注册包到 Packagist
composer config repositories.my-package path /path/to/package
composer require vendor/my-package:@dev
```

---

# **八、常用插件推荐**
1. **hirak/prestissimo** - 并行下载加速（Composer 2+ 已内置）
2. **phpstan/phpstan** - PHP 静态分析
3. **friendsofphp/php-cs-fixer** - 代码风格检查
4. **nunomaduro/phpinsights** - 代码质量分析

安装插件：
```bash
composer global require phpstan/phpstan
```
