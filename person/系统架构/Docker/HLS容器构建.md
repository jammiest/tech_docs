# PHP容器构建

## 目录

构建目录结构
```txt
|--static/
   |--m3u8/
   |--mp4/
   |--mp4_finished/
   |--convert.sh
|--dockerfile
```

> 其中m3u8，mp4，mp4_finished都为空目录

## convert.sh

```shell
#!/bin/bash
#convert mp4 to ts
function batch_convert() {
    for file in `ls "$1"|tr " " "?"`
    do
        file=${file//\?/ }
	echo $1"/$file"
        if [ -d $1"/$file" ]
        then
            batch_convert $1"/$file"
        elif [ "${file##*.}" = "mp4" -o "${file##*.}" = "MP4" ]
        then
            #replace old path, remove suffix
            file2folder="$TS_VIDEO_DIR${1/$MP4_VIDEO_DIR/}/${file%.*}"
	    echo $file2folder
            if [ ! -d "$file2folder" ]
            then
                mkdir -p "$file2folder"
            fi
            echo $1" : "$1"/$file"
            ffmpeg -i $1"/$file" -f segment -segment_time $VIDEO_SEGMENT_TIME -segment_format mpegts -segment_list "$file2folder/playlist.m3u8" -c copy -bsf:v h264_mp4toannexb -map 0 "$file2folder/%03d.ts" >> /dev/null
            if [ 0 -eq $? ]
            then
                mv $1"/$file" /root/mp4_finished/
            fi
            echo $file $file2folder >> $MP4_VIDEO_DIR/log
        fi
    done
}
batch_convert $MP4_VIDEO_DIR

```

## dockerfile文件

```dockfile
FROM alpine:latest

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

COPY static/ /root/

WORKDIR /root

ENV MP4_VIDEO_DIR=/root/mp4 \
    TS_VIDEO_DIR=/root/m3u8 \
    VIDEO_SEGMENT_TIME=5

RUN apk update && apk add ffmpeg bash vim && rm -rf /var/cache/apk/*

# ENTRYPOINT ["bash","convert.sh"]
```

## 构建镜像

```shell
docker build -t m3u8-gen-cron .
```

## 创建容器

```shell
docker run -dit --name m3u8-gen-cron -v /mnt/www/mp4:/root/mp4 -v /mnt/www/m3u8:/root/m3u8 -v /mnt/www/mp4_finished:/root/mp4_finished m3u8-gen-cron:latest
```

## 使用

```shell
docker exec -it m3u8-gen-cron bash -c './convert'
```

> 该操作会自动把挂载到容器里的mp4目录里的视频文件 转化为m3u8格式的文件到m3u8目录中，同时迁移该mp4文件到mp4_finished目录里，
> 该容器支持批量操作