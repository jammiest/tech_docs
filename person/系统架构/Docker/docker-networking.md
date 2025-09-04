# Network网络管理

> 官方参照文档 <https://docs.docker.com/engine/reference/commandline/network/>


| Command                   | Description              |
| ------------------------- | ------------------------ |
| `docker network connect`    | 连接一个容器到一个网络上 |
| `docker network create`     | 创建一个网络             |
| `docker network disconnect` | 从一个网络上断开一个容器 |
| `docker network inspect`    | 显示一个或多个网络信息   |
| `docker network ls`         | 列出所有的网络           |
| `docker network prune`      | 移除未被使用的网络       |
| `docker network rm`         | 移除一个或多个网络       |

## docker network create

创建网络

## docker network connect

> 将一个容器连接到制定的网络上

### 用法

`docker network create [OPTIONS] NETWORK`

### 用法

```shell
docker network connect [OPTIONS] NETWORK CONTAINER
```

Options

| 选项明          | 默认 | 说明                                       |
| --------------- | ---- | ------------------------------------------ |
| --alias         |      | Add network-scoped alias for the container |
| --driver-opt    |      | 为这个网络添加驱动                         |
| --ip            |      | IP4地址 (如 172.30.100.104)                |
| --ip6           |      | IP6地址 (如 2001:db8::33)                  |
| --link          |      | 添加一个连接到另一个容器                   |
| --link-local-ip |      | 为这个容器添加一个本地连接地址             |

### 示例

连接一个运行中的容器到一个网络上

`docker network connect multi-host-network container1`

连接一个容器到一个网络上，当容器被启动时

当容器被第一次启动时，你也可以使用选项`docker run --network=<network-name>`进行快速配置

`docker run -itd --network=multi-host-network busybox`

