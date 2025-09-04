# PHP 源码编译指南

PHP 源码编译允许你自定义 PHP 的安装选项和扩展，特别适合需要特定配置或优化性能的场景。

## 1. 准备工作

### 1.1 安装编译工具和依赖

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y build-essential autoconf bison re2c \
    libxml2-dev libsqlite3-dev libssl-dev zlib1g-dev \
    libcurl4-openssl-dev libonig-dev libreadline-dev

# CentOS/RHEL
sudo yum groupinstall -y "Development Tools"
sudo yum install -y autoconf bison re2c libxml2-devel \
    sqlite-devel openssl-devel zlib-devel libcurl-devel \
    oniguruma-devel readline-devel
```

### 1.2 下载 PHP 源码

```bash
# 从官方下载
wget https://www.php.net/distributions/php-8.2.0.tar.gz
tar -xzvf php-8.2.0.tar.gz
cd php-8.2.0
```

或者从 Git 获取最新开发版本：

```bash
git clone https://github.com/php/php-src.git
cd php-src
git checkout PHP-8.2
```

## 2. 配置编译选项

### 2.1 基本配置命令

```bash
./buildconf --force
./configure --help  # 查看所有可用选项
```

### 2.2 常用配置选项

```bash
./configure \
    --prefix=/usr/local/php8.2 \
    --with-config-file-path=/usr/local/php8.2/etc \
    --enable-fpm \
    --with-fpm-user=www-data \
    --with-fpm-group=www-data \
    --enable-mbstring \
    --enable-opcache \
    --enable-pcntl \
    --enable-sockets \
    --enable-bcmath \
    --enable-intl \
    --with-zlib \
    --with-curl \
    --with-openssl \
    --with-readline \
    --with-pdo-mysql \
    --with-mysqli \
    --with-pear
```

### 2.3 常见扩展配置

| 扩展 | 配置选项 |
|------|----------|
| GD | `--with-gd --with-freetype --with-jpeg` |
| PostgreSQL | `--with-pdo-pgsql --with-pgsql` |
| SQLite3 | `--with-sqlite3 --with-pdo-sqlite` |
| Redis | 需单独安装 pecl 扩展 |
| Xdebug | 需单独安装 pecl 扩展 |

## 3. 编译和安装

```bash
# 编译 (使用多核加速)
make -j$(nproc)

# 安装
sudo make install

# 安装后检查
/usr/local/php8.2/bin/php -v
```

## 4. 配置文件

### 4.1 复制配置文件

```bash
sudo cp php.ini-development /usr/local/php8.2/etc/php.ini
sudo cp sapi/fpm/php-fpm.conf /usr/local/php8.2/etc/php-fpm.conf
sudo cp sapi/fpm/www.conf /usr/local/php8.2/etc/php-fpm.d/www.conf
```

### 4.2 常用 php.ini 设置

```ini
; 错误报告
error_reporting = E_ALL
display_errors = On

; 时区
date.timezone = "Asia/Shanghai"

; 上传限制
upload_max_filesize = 64M
post_max_size = 64M

; OPcache 配置
opcache.enable=1
opcache.memory_consumption=128
opcache.max_accelerated_files=4000
opcache.validate_timestamps=60
```

## 5. PHP-FPM 配置

### 5.1 基本配置

```ini
[www]
user = www-data
group = www-data
listen = 127.0.0.1:9000
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
```

### 5.2 创建 systemd 服务

创建 `/etc/systemd/system/php-fpm.service`：

```ini
[Unit]
Description=PHP FastCGI Process Manager
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/php8.2/sbin/php-fpm --nodaemonize --fpm-config /usr/local/php8.2/etc/php-fpm.conf
ExecStop=/bin/kill -QUIT $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

然后启用服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```

## 6. 添加环境变量

编辑 `~/.bashrc` 或 `/etc/profile`：

```bash
export PATH="/usr/local/php8.2/bin:$PATH"
```

然后执行：

```bash
source ~/.bashrc
```

## 7. 常见问题解决

### 7.1 编译错误处理

1. **缺少依赖**：根据错误信息安装对应的开发包
2. **内存不足**：增加 swap 空间或使用 `-j2` 减少并行编译任务
3. **权限问题**：确保对安装目录有写权限

### 7.2 扩展安装

```bash
# 使用 pecl 安装扩展
pecl install redis
pecl install xdebug

# 然后在 php.ini 中添加
extension=redis.so
zend_extension=xdebug.so
```

### 7.3 多版本共存

可以使用 `update-alternatives` 管理多个 PHP 版本：

```bash
sudo update-alternatives --install /usr/bin/php php /usr/local/php8.2/bin/php 82
sudo update-alternatives --install /usr/bin/php php /usr/local/php7.4/bin/php 74
sudo update-alternatives --config php
```

## 8. 性能优化编译选项

```bash
./configure \
    --enable-inline-optimization \
    --enable-opcache \
    --enable-opcache-jit \
    --with-zend-vm=CALL \
    --disable-debug \
    --disable-rpath
```

## 9. 编译参数参考表

| 参数 | 说明 |
|------|------|
| `--prefix` | 安装目录 |
| `--enable-fpm` | 启用 PHP-FPM |
| `--with-config-file-path` | php.ini 目录 |
| `--disable-all` | 禁用所有默认扩展 |
| `--enable-cli` | 启用 CLI SAPI |
| `--enable-cgi` | 启用 CGI SAPI |
| `--enable-embed` | 启用嵌入式 SAPI |
| `--enable-debug` | 启用调试符号 |
| `--with-libdir=lib64` | 64位系统库目录 |

## 10. 卸载编译安装的 PHP

```bash
sudo make uninstall
sudo rm -rf /usr/local/php8.2
```

通过源码编译安装 PHP 可以获得更好的性能和定制性，特别适合生产环境部署和特定需求场景。