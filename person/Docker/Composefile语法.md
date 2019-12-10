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
