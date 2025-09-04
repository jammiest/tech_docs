# FastDFS容器构建

参考：<https://hub.docker.com/r/moocu/fastdfs>

## 创建并进入目录

```sh
mkdir .docker-fastdfs
cd .docker-fastdfs
```

## FastDFS源码下载

源码位置：  
FastDFS ：<https://github.com/happyfish100/fastdfs/releases>  
libfastcommon ：<https://github.com/happyfish100/libfastcommon/releases>

```sh
wget https://github.com/happyfish100/fastdfs/releases -O FastDFS-V5.11.tar.gz
wget libfastcommon-V1.0.39.tar.gz
```

## Dockerfile

```dockerfile
FROM debian:stretch-slim

ADD FastDFS-V5.11.tar.gz /usr/local/
ADD libfastcommon-V1.0.39.tar.gz /usr/local/

RUN apt-get update
RUN apt-get -y install make cmake gcc gcc-6

RUN set -ex; \
	cd /usr/local/libfastcommon-V1.0.39; \
        ./make.sh && ./make.sh install

RUN set -ex; \
        cd /usr/local/FastDFS-V5.11; \
        ./make.sh && ./make.sh install

ENV FASTDFS_VERSION 5.11
ENV FASTDFS_TRACKER tracker
ENV FASTDFS_STORAGE storage
ENV SERVER storage

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]
```

## 构建镜像

```sh
docker build -t my_fastdfs .
```


## 构建容器

### 构建tracker容器
> 构建tracker容器

```sh
docker run -e SERVER=tracker -v /data/etc/fdfs:/etc/fdfs -v /data/fastdfs/tracker:/fastdfs/tracker -p 22122:22122 --name tracker -d my_fastdfs
 ```

其中`/data/etc/fdfs`为FastDFS配置目录，可根据自己实际情况进行调整，`/data/fastdfs/tracker`目录为FastDFS tracker保存路径

### 构建storage容器

>  构建storage容器

```sh
docker run -v /data/etc/fdfs:/etc/fdfs -v /data/fastdfs/storage:/fastdfs/storage -p 23000:23000 -p 8888:8888 --name storage -d my_fastdfs
 ```

其中`/data/etc/fdfs`为FastDFS配置目录，可根据自己实际情况进行调整，`/data/fastdfs/storage`目录为FastDFS文件数据存储路径

### 构建tracker+storage容器

```sh
docker run -e SERVER=all -v /data/etc/fdfs:/etc/fdfs -v /data/fastdfs/storage:/fastdfs/storage -v /data/fastdfs/tracker:/fastdfs/tracker -p 22122:22122 -p 23000:23000 -p 8888:8888 --name storage -d my_fastdfs
 ```

其中`/data/etc/fdfs`为FastDFS配置目录，可根据自己实际情况进行调整，`/data/fastdfs/storage`目录为FastDFS文件数据存储路径，`/data/fastdfs/storage`目录为FastDFS文件数据存储路径
