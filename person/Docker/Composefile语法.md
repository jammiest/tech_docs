# Docker

## build

构建容器的上下文

语法

- 字符串

```dockerfile
version: "3.7"
services:
  webapp:
    build: ./dir
```

- 包含context,Dockerfile,args的对象

```dockerfile
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

- 如果你同时使用`image`和`build`，构建后的镜像将被命名为`image`所对应的镜像名和TAG标签。

```dockerfile
build: ./dir
image: webapp:tag
```

## CONTEXT

上下文，包含Dockerfile文件的目录或者git存储库的URL地址

当这个值被定义为相对路径时，为相对composefile文件的相对地址，此目录也是发送到Docker守护程序的构建上下文。

```dockerfile
build:
  context: ./dir
```

## DOCKERFILE

指定编排容器的dockerfile文件，同时构建路径也必须要被定义

```composefile
build:
  context: .
  dockerfile: Dockerfile-alternate
```

## ARGS

添加在构建过程中访问的环境变量参数。

1、首先再你对应的dockerfile中定义

```dockerfile
ARG buildno
ARG gitcommithash

RUN echo "Build number: $buildno"
RUN echo "Based on commit: $gitcommithash"
```

2、然后再composefile中的`build`结构中定义`args`，可用以mapping或list的格式来定义

```docker-compose
build:
  context: .
  args:
    buildno: 1
    gitcommithash: cdc3b19
```

```docker-compose
build:
  context: .
  args:
    - buildno=1
    - gitcommithash=cdc3b19
```

## CACHE_FROM

docker引擎用于缓存解析的镜像列表。

```docker-compose
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```

## LABELS

使用Docker标签将元数据添加到生成的镜像中。您可以使用数组或字典。

```docker-compose
 build:
  context: .
  labels:
    com.example.description: "Accounting webapp"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""
```


```docker-compose
build:
  context: .
  labels:
    - "com.example.description=Accounting webapp"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
```

## SHM_SIZE

设置此构建容器的`/dev/shm`分区的大小。指定为表示字节数的整数值或表示字节值的字符串。

```docker-compose
build:
  context: .
  shm_size: '2gb'
```

```docker-compose
build:
  context: .
  shm_size: 10000000
```

## TARGET

根据Dockerfile中的定义构建指定的阶段。有关详细信息，请参见[多步构建文档](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)。

```docker-compose
build:
  context: .
  target: prod
```

## cap_add, cap_drop

添加或删除容器功能。有关完整列表，请参见`man 7 capabilities`。

```docker-compose
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

## cgroup_parent

指定一个容器可选的父级cgroup。

```docker-compose
cgroup_parent: m-executor-abcd
```

## command

```docker-compose
command: bundle exec thin -p 3000
```

```docker-compose
command: ["bundle", "exec", "thin", "-p", "3000"]
```

## configs

授权某个服务可以访问它下面配置的configs，支持两种语法

### 短语法

短语法只指定config名称，授权容器可以访问config，并将其挂载到该容器下的`/<config_name>`

下面的例子授权redis服务访问`my_config`和`my_other_config`配置。`my_config`的值设置的是`./my_config.txt`，而`my_other_config`的值指定的是外部资源，这就意味着该值已经被定义在Docker中了。通过运行`docker config create`命令或其他堆栈部署。如果外部配置不存在，则堆栈部署将失败，并显示“找不到配置”错误。

```docker-compose
version: "3.7"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

### 长语法

长语法提供了更细粒度的控制

- `source`  ：config的名称
- `target`  ：被挂载到容器后的文件名称，默认是`/<source>`
- `uid`和`gid`  ：被挂载到容器的文件的所有者和所属组ID
- `mode`  ：被挂载到容器中的文件的权限（PS：如果你不熟悉UNIX的权限模式，可以用这个工具 http://permissions-calculator.org）

下面这个例子将在容器下设置`my_config`和`redis_config`，设置权限是`0440`，所有者和所属组都是`103`，`redis`服务不可以访问`my_other_config`配置

```docker-compose
version: "3.7"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

*您可以授予服务访问多个配置的权限，并且可以混合长短语法。定义配置并不意味着授予对它的服务访问权限。*

## container_name

自定义容器名称，而不是用默认生成的名称

```docker-compose
container_name: my-web-container
```

!> 由于Docker容器名称必须唯一，因此如果您指定了自定义名称，则不能将服务扩展到超过1个容器。尝试这样做会导致错误。

!> Note: 这个参数会被swarm模式忽略。

## depends_on

表示服务之间的依赖关系，服务依赖关系导致以下行为：

- `docker-compose up`  按照依赖顺序启动服务
- `docker-compose up SERVICE`  自动包含服务的依赖
- `docker-compose stop`  按照依赖顺序停止服务

下面的例子中，`db`和`redis`会先于`web`启动，启动`web`的时候也会创建并启动`db`和`redis`，`web`停止之前会先停止`db`和`redis`

```docker-compose
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

注意：`depends_on`不会等待`db`和`redis`启动好了再启动`web`

## deploy

指定与服务的部署和运行有关的配置。这仅在通过[docker stack deploy](https://docs.docker.com/engine/reference/commandline/stack_deploy/)部署到集群时才生效，并且被`docker-compose up`和`docker-compose run`忽略。

```docker-compose
version: "3.7"
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

## ENDPOINT_MODE

为连接到群集的外部客户端指定服务发现方法。

- `endpoint_mode: vip` - Docker为服务分配了一个虚拟IP（VIP），该虚拟IP充当客户端访问网络上服务的前端。 Docker在客户端和服务的可用工作节点之间路由请求，而无需客户端知道有多少节点正在参与服务或其IP地址或端口。 （这是默认设置。
- `endpoint_mode: dnsrr` - DNS轮询（DNSRR）服务发现不使用单个虚拟IP。 Docker设置服务的DNS条目，以便对服务名称的DNS查询返回IP地址列表，并且客户端直接连接到其中之一。在想要使用自己的负载平衡器或混合Windows和Linux应用程序的情况下，DNS轮询很有用。

```docker-compose
version: "3.7"

services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    networks:
      - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip

  mysql:
    image: mysql
    volumes:
       - db-data:/var/lib/mysql/data
    networks:
       - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

volumes:
  db-data:

networks:
  overlay:
```

## LABELS

指定服务标签。这些标签仅在服务上设置，而不在服务的任何容器上设置。

```docker-compose
version: "3.7"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```

为了在容器上设置标签，通过在`deploy`外部使用`labels`

```docker-compose
version: "3.7"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```

## MODE

`global`（每个集群节点只有一个容器） 或者 `replicated` （指定数量的容器）。默认是 `replicated`

```docker-compose
version: "3.7"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

## PLACEMENT

指定约束和首选项的位置。有关语法以及约束和首选项的可用类型的完整说明，请参阅docker服务create文档。

```docker-compose
version: "3.7"
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - engine.labels.operatingsystem == ubuntu 14.04
        preferences:
          - spread: node.labels.zone
```

## REPLICAS

如果服务被定义成`replicated`（这是默认设置）, 指定在任何给定时间应运行的容器数。

```docker-compose
version: "3.7"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
```

## RESOURCES

配置容器的资源

在此一般示例中，`redis`服务被限制为使用不超过 50M 的内存和`0.50`（单核的50％）的可用处理时间（CPU），并具有`20M`的内存和`0.25`的CPU时间保留（相对于总是可用的情况）

```docker-compose
version: "3.7"
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

## RESTART_POLICY

配置是否以及如何在退出容器时重新启动容器。替换为重新启动。

```docker-compose
version: "3.7"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

## dns

自定义的DNS服务器

```docker-compose
dns: 8.8.8.8
```

```docker-compose
dns:
  - 8.8.8.8
  - 9.9.9.9
```

## dns_search

自定义的DNS搜索域名


```docker-compose
dns_search: example.com
```

```docker-compose
dns_search:
  - dc1.example.com
  - dc2.example.com
```

## entrypoint

```docker-compose
entrypoint: /code/entrypoint.sh
```

```docker-compose
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

## env_file

从一个文件中添加环境变量

如果您使用docker-compose -f FILE指定了Compose文件，则env_file中的路径相对于该文件所在的目录。

在环境部分中声明的环境变量将覆盖这些值-即使这些值为空或未定义，这也适用。多配置文件同变量将会被覆盖

```docker-compose
env_file: .env
```

```docker-compose
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

Compose期望环境文件中的每一行均为`VAR=VAL`格式。以`＃`开头的行被视为注释，并被忽略。空行也将被忽略。

```docker-compose
# Set Rails/Rack environment
RACK_ENV=development
```

## environment

添加环境变量。您可以使用数组或字典。任何布尔值； True，false，yes否，需要用引号引起来，以确保YML解析器不会将其转换为True或False。

仅具有键的环境变量在运行Compose的计算机上解析为它们的值，这对于秘密或特定于主机的值很有用。

```docker-compose
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
```

```docker-compose
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

## expose

公开端口而不将其发布到主机上-只有链接的服务才能访问它们。只能指定内部端口。

```docker-compsoe
expose:
 - "3000"
 - "8000"
```

## external_links

## image

指定要从中启动容器的图像。可以是存储库/Tag或部分image ID。

## logging

服务的日志记录配置。

```docker-compose
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

`driver`指定了服务容器的日志记录驱动程序，以及docker run的`--log-driver`选项[更多参见](https://docs.docker.com/engine/admin/logging/overview/)。

这个默认值时json格式的文件

```docker-compose
driver: "json-file"
driver: "syslog"
driver: "none"
```

Logging `options`时键值对，`syslog`的demo如下：

```docker-compose
driver: "syslog"
options:
  syslog-address: "tcp://192.168.0.42:123"
```
默认驱动程序json-file具有限制存储日志量的选项。为此，请使用键值对以获取最大存储大小和最大文件数：

```docker-compose
options:
  max-size: "200k"
  max-file: "10"
```

下面时`docker-compose.yml`限制日志存储的一个示例

```docker-compose
version: "3.7"
services:
  some-service:
    image: some-service
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

## network_mode

网络模式

```docker-compose
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

## networks

连接的网络

```docker-compose
services:
  some-service:
    networks:
     - some-network
     - other-network
```

## ALIASES

这个服务在网络上（相对于主机名）的别名，同一网络上的其他容器可以使用服务名称或此别名连接到服务的容器之一。

由于别名是网络范围的，因此同一服务在不同的网络上可以具有不同的别名。

```docker-compose
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```

## IPV4_ADDRESS, IPV6_ADDRESS

## pid

## ports

暴露的端口

### 短语法

```docker-compose
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```

### 长语法

长格式语法允许配置其他不能以短格式表示的字段。

- `target`: 容器内的端口
- `published`: 公共暴露的端口
-`protocol`: 端口协议 (`tcp` or `udp`)
- `mode`: `host` 用于在每个节点上发布主机端口, 或者 `ingress` 群模式端口以实现负载平衡。

## restart

## volumes

```docker-compose
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```


### 完整示例

```docker-compose
version: "3.7"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```
