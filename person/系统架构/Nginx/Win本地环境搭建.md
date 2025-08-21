# Windows配置nginx

1、官方教程<http://nginx.org/en/docs/windows.html>

2、下载nginx压缩包[nginx/Windows-1.15.12](http://nginx.org/download/nginx-1.15.12.zip)

3、解压文件，进入目录中执行

```cmd
start nginx
```

4、查看nginx进程
```cmd
TASKLIST /FI "imagename eq nginx.exe"
```

常用命令

```cmd
nginx -s stop	快速关闭
nginx -s quit	优雅的方式关闭
nginx -s reload	重启
nginx -s reopen 重新打开日志文件
```