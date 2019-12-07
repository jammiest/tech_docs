# mysql 容器构建

```shell
# 容器名自定义为mysql5.7.26
# 端口暴露：3306 <-> 3306
# 目录挂载：(外部)/data/docker/mysql/conf.d:/etc/mysql/conf.d（内部）：配置文件
# 目录挂载：(外部)/mnt/windows/data/mysql:/var/lib/mysql（内部）:数据
docker run --name mysql5.7.26 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v /data/docker/mysql/conf.d:/etc/mysql/conf.d -v /mnt/windows/data/mysql:/var/lib/mysql -d mysql:5.7.26
```

