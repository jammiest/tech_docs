# docker远程xdebug调试配置

> xdebug 配置以vscode编辑器配置为例

## 进入服务端docker中的xdebug配置

- 安装最高支持php环境的xdebug版本：pecl install xdebug-2.5.4,并且添加配置文件
  
```bash
docker exec -it php bash
pecl install xdebug-2.5.4
echo -e 'zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so\nxdebug.idekey=VSCODE\nxdebug.remote_connect_back=1\nxdebug.profiler_enable=1\nxdebug.remote_autostart=On\nxdebug.remote_enable=On' >> /usr/local/etc/php/conf.d/xdebug.ini
```

重启php-fpm：

```bash
kill -USR2 1
```

## chrome浏览器配置

> 下载debug插件
>
> <https://chrome.google.com/extensions/detail/eadndfjplgieldjbigjakmdgkmoaaaoc>
>
> 启用插件调试功能

## Postman调试配置

> 在请求中添加cookie：

```http
XDEBUG_SESSION=VSCODE; path=/; domain=.lumen.com; Expires=Tue, 19 Jan 2038 03:14:07 GMT;
```

## IDE设置

1、IDE配置xdebug和远程目录映射【vscode为例：launch.json】：

```json
{
    "name": "Listen for XDebug",
    "type": "php",
    "request": "launch",
    "port": 9000,
    "pathMappings": {
    "/var/local/www/myproject": "${workspaceRoot}"
    },
},
```

## xdebug性能分析工具webgrind

> 下载 <https://github.com/jokkedk/webgrind>