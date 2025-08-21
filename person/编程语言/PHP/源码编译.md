# PHP7源码编译示例

## 下载解压

```shell
wget https://www.php.net/distributions/php-7.3.5.tar.gz
tar -xvf php-7.3.5.tar.gz
cd php-7.3.5
```

## 安装依赖

```shell
yum -y install libxml2 libxml2-devel openssl openssl-devel curl-devel libjpeg-devel libpng-devel freetype-devel
```

## 配置参考

参考：<https://www.php.net/manual/zh/configure.about.php>
或帮助指令：`./configure --help`

```shell
./buildconf --force

./configure \
--prefix=/usr/local/php7.3.5 \                              [PHP7安装的根目录]
--exec-prefix=/usr/local/php7.3.5 \
--bindir=/usr/local/php7.3.5/bin \
--sbindir=/usr/local/php7.3.5/sbin \
--includedir=/usr/local/php7.3.5/include \
--libdir=/usr/local/php7.3.5/lib/php \
--mandir=/usr/local/php7.3.5/php/man \
--with-config-file-path=/usr/local/php7.3.5/etc \           [PHP7的配置目录]
--with-mysql-sock=/var/run/mysql/mysql.sock \           [PHP7的Unix socket通信文件]
--with-mcrypt=/usr/include \
--with-mhash \
--with-openssl \
--with-mysql=shared,mysqlnd \                           [PHP7依赖mysql库]              
--with-mysqli=shared,mysqlnd \                          [PHP7依赖mysql库]
--with-pdo-mysql=shared,mysqlnd \                       [PHP7依赖mysql库]
--with-gd \
--with-iconv \
--with-zlib \
--enable-zip \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared \
--enable-xml \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-mbregex \
--enable-mbstring \
--enable-ftp \
--enable-gd-native-ttf \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-soap \
--without-pear \
--with-gettext \
--enable-session \                                      [允许php会话session]
--with-curl \                                           [允许curl扩展]
--with-jpeg-dir \
--with-freetype-dir \
--enable-opcache \                                      [使用opcache缓存]
--enable-fpm \
--enable-fastcgi \
--with-fpm-user=nginx \                                 [php-fpm的用户]
--with-fpm-group=nginx \                                [php-fpm的用户组]
--without-gdbm \
--with-mcrypt=/usr/local/related/libmcrypt \            [指定libmcrypt位置]
--disable-fileinfo
```

## 编译

```shell
make clean && make && make install
```

## 测试(非必选)

```shell
make test
```

## 添加路径（非必选）

打开文件

```shell
vim /etc/profile
```

在文件底部新增如下内容 [ /etc/profile ] ：

```/etc/profile
PATH=$PATH:/usr/local/apache2/php/bin
export PATH
```

执行当前shell配置脚本文件

```shell
source /etc/profile
```

## 查看php版本信息

```shell
php -v
-------------------
# PHP 7.3.5 (cli) (built: May 15 2019 14:14:04) ( NTS )
# Copyright (c) 1997-2018 The PHP Group
# Zend Engine v3.3.5, Copyright (c) 1998-2018 Zend Technologies
```

## 复制配置文件

```shell
# 将源码中的配置文件复制到安装目录配置目录下
cp php-7.3.5/php.ini* /usr/local/php7.3.5/etc/
cp /usr/local/php7.3.5/etc/php.ini-development /usr/local/php7.3.5/etc/php.ini
```

## 安装pecl

参考：<https://pear.php.net/manual/en/installation.getting.php>

```shell
wget http://pear.php.net/go-pear.phar
php go-pear.phar
```

## 配置pecl

```
pear config-set php_ini /usr/local/php7.3.5/etc/php.ini
pecl config-set php_ini /usr/local/php7.3.5/etc/php.ini
```