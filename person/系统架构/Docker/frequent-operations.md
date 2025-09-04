# Docker基本指令预览


## 启动容器并进入Bash终端

使用`docker run`命令启动一个新的容器，并通过-it参数分配伪终端并保持标准输入开放：

```bash
docker run -it my_container /bin/bash
```

## 创建自定义网络

使用docker network create命令创建一个自定义网络，并指定子网和网关。例如：

```docker network create --driver bridge --subnet 172.22.1.0/24 --gateway 172.22.1.1 my_net```
在这个命令中：

`--driver bridge` 指定网络驱动类型为桥接。

`--subnet 172.22.1.0/24` 指定子网。

`--gateway 172.22.1.1` 指定网关。

`my_net` 是自定义网络的名称。

## 在自定义网络中运行容器

使用docker run命令在自定义网络中运行容器，并为其指定静态IP地址。例如：

```docker run -it --net my_net --ip 172.22.1.100 --name my_container ubuntu:latest /bin/bash```
在这个命令中：

`--net my_net3` 指定容器连接到自定义网络my_net3。

`--ip 172.22.1.100` 指定容器的静态IP地址。

`--name my_container` 指定容器的名称。

`ubuntu:latest` 是容器使用的镜像名称。

## 使用docker-compose

如果你使用docker-compose来管理容器，可以在docker-compose.yml文件中指定网络配置。例如：

```docker
version: '2'
services:
my_service:
image: ubuntu:latest
container_name: my_container
networks:
my_net:
ipv4_address: 172.22.1.100

networks:
my_net:
external: true
```

在这个配置文件中：

`networks` 部分定义了一个外部网络my_net3。

`my_service` 服务连接到my_net3网络，并指定了静态IP地址`172.22.1.100`。