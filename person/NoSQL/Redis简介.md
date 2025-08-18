# Redis

> 参考文档  
> 官方英文文档：<https://redis.io/documentation>  
> 官方中文文档：<http://www.redis.cn/documentation.html>  
> 中文文档[腾讯]：<https://cloud.tencent.com/developer/doc/1203>

## 安装

环境：
操作系统：CentOS 7 minimal
工具：wget,gcc

1、下载解压编译

```shell
# 下载（需要已经安装wget）
wget http://download.redis.io/releases/redis-5.0.4.tar.gz

# sha256值校验
sha256sum redis-5.0.4.tar.gz

# 解压
tar zxf redis-5.0.4.tar.gz

# 移动并且进入目录
mv redis-5.0.4 /usr/local
cd /usr/local/redis-5.0.4

# 要清除所有生成的文件
make distclean

# 编译（需要已经安装gcc）
make
```

2、启动Redis服务器

```shell
/usr/local/redis-5.0.4/src/redis-server
```

3、内置的客户端连接服务端

```shell
src/redis-cli

-----------
redis> set foo bar
OK
redis> get foo
"bar"
------------
```