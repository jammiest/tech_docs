# Docsify容器构建

环境：Linux,Docker

> 1. 新建目录，然后新建文件Dockerfile，内容如下

```Dockerfile
# Use node 4.4.5 LTS
FROM node:4.4.5
ENV LAST_UPDATED 20160605T165400

# Copy source code
COPY . /app

# Change working directory
WORKDIR /app

# Install dependencies
RUN npm i marked prismjs docsify-cli -g

# Expose API port to the outside，容器对外暴露的端口号
EXPOSE 3000

# Launch application
# /var/local/www/jyj_docs为映射到容器内的docsify路径
CMD cd /var/local/www/jyj_docs && docsify serve
```

> 再当前目录下执行构建指令

```shell
# 调整镜像容器一键指令
# docker rm jyj_docs && docker rmi jyj_docs && docker build -t jyj_docs . && docker run -it --name jyj_docs -p 3000:3000 -v /mnt/windows:/var/local/www jyj_docs

# 构建镜像 && 实例化并且启动容器 
# 镜像和容器名自定义为：jyj_docs 
# 并且将容器3000内端，映射给本机的3000端口，使得外部访问本机的3000端口能访问到容器内部
# 将主机的目录（/mnt/windows）映射到容器里的（/var/local/www）上
docker build -t jyj_docs . && docker run -it --name jyj_docs -p 3000:3000 -v /mnt/windows:/var/local/www jyj_docs

# 删除容器 && 删除镜像
# 镜像和容器名自定义为：jyj_docs
docker rm jyj_docs && docker rmi jyj_docs && docker build -t jyj_docs .
```