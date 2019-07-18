# Nginx容器构建

*参考：*  
*docker build指令 <https://docs.docker.com/engine/reference/run/>*  
*nginx容器：<https://hub.docker.com/_/nginx>*

> 环境说明：CentOS7 minimal

## 通用快速构建容器

```shell
# 参数说明：
#   容器别名：--name nginx：构建容器名为nginx的容器
#   端口暴露：容器内部80映射到服务器的80端口
#   目录挂载：外部的/var/www路径挂载到docker nginx容器中的/var/www中
#   配置目录：外部的/etc/docker/nginx/conf.d 映射到容器内部的/etc/nginx/conf.d中
docker run -it -d --name nginx -p 80:80 -v /var/www:/var/www -v /etc/docker/nginx/conf.d:/etc/nginx/conf.d nginx

# 查看所有容器
docker ps -a
```

> 该Nginx容器与其他容器完全独立，与其他Docker CGI的容器访问，需要这类容器对外暴露端口号（如在run构建这类容器时，需要-p 9000:9000），故该Nginx容器通过ip+端口号发送请求

Nginx虚拟目录配置示例

```conf
server {
	listen 80;
	listen [::]:80;
	server_name novel.03pl.local;
	location / {
		proxy_pass http://192.168.0.240:3000;
	}
}
```

## 关联式Nginx容器构建

> 以 nginx + docsify 配置为例

  前提条件：已经创建了一个docsify的容器，且该容器正在运行

```shell
# 关联的容器名为docsify
docker run -it -d --name nginx --link docsify -p 80:80 -v /var/www:/var/www -v /etc/docker/nginx/conf.d:/etc/nginx/conf.d nginx

# 查看所有容器
docker ps -a
```

> 该Nginx容器与其他容器关联，可通过多种方式访问被关联的容器：
- 通过被关联容器对外暴露的端口号方式访问，故该Nginx容器通过ip+端口号发送请求：如192.168.10.240:3000
- 通过被关联容器的套接字访问：如http://docsify:3000

配置文件示例

```conf
server {
	listen 80;
	listen [::]:80;
	server_name novel.03pl.local;
	location / {
		proxy_pass http://docsify:3000; # 或者设置为 http://192.168.0.240:3000
	}
}
```

## Nginx容器管理

```shell
docker start nginx
docker stop nginx
docker restart nginx

# 实时查看nginx容器请求日志
docker logs --tail 10 -f nginx
```

## 题外话

> PS.