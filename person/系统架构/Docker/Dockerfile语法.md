# Dockerfile语法

## FROM

语法

- `FROM <image> [AS <name>]`
- `FROM <image>[:<tag>] [AS <name>]`
- `FROM <image>[@<digest>] [AS <name>]`

`FROM`指令是一个新的初始化构建指令和为后续指令设置基本的镜像，因此一个可用的*dockerfile*文件必须是`FROM`指令开始的，这个镜像可以是任何可用的镜像。

- `ARG`是`FROM`之前的唯一指令
- `FROM`能够出现多次在一个单独的`dockerfile`通常是为了构建多镜像或一个镜像的构建依赖于另一个镜像
- 可选的参数**name**能够给该镜像命名，这个**name**能够为后续的`FROM`或`COPY --from=<name|index>`使用。

## ARG

`FROM` 能够使用 `ARG` 指令定义的变量，只要该`ARG`在`FROM`之前定义即可。

```dockerfile
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

一个`ARG`指令的声明是在`FROM`之前和一个构建步骤之外的，因此在`FROM`步骤之后的任何指令都无法再使用它。为了使用这个在`FROM`之前声明的变量，就必须通过`ARG`只能声明一个不带指的改变量才能使用。如

```dockerfile
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

## RUN

语法

- `RUN <command>`
- `RUN ["executable", "param1", "param2"]`

## CMD

语法

- `CMD ["executable","param1","param2"]` （exec风格，首选的格式）
- `CMD ["param1","param2"]`  （作为`ENTRYPOINT`默认的参数）
- `CMD command param1 param2` (shell风格)

Dockerfile只有最后一个`CMD`指令能生效

## EXPOSE

端口监听

语法

- `EXPOSE <port> [<port>/<protocol>...]` 默认的`protocol`是tcp

示例1

EXPOSE默认使用的是tcp协议，你也可以定义UDP

```dockerfile
EXPOSE 80/udp
```

为了同一端口同时监听TCP和UDP的数据，可以定义如下两行：

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

## ENV

环境变量设置

语法

- `ENV variable value`

支持的指令

- `ADD`
- `COPY`
- `ENV`
- `EXPOSE`
- `FROM`
- `LABEL`
- `STOPSIGNAL`
- `USER`
- `VOLUME`
- `WORKDIR`

用法

```dockerfile
FROM busybox
ENV foo /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux
```

`ENV`支持批量赋值和变量赋值，如

```dockerfile
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```

结果 `def`被设置为`hello`, 而不是`bye`。  
然而`ghi` 被设置为`bye`，因为它被设置为和环境变量`abc`一样的值`bye`。

## ADD

语法

- `ADD [--chown=<user>:<group>] <src>... <dest>`
- `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` 该格式支持包含空格情况

`ADD`指令是用于拷贝从`<src>`多文件，多目录，或远程URLs的文件到到当前镜像的文件系统<dest>目录下,目录本身不会被复制，只有内容被复制

如果`<src>`是能被是别的压缩包格式（identity,gzip,bzip2或xz），将会进行解压成为文件夹

示例1

`<src>`能够包含通配符，语法是`Go`语言的文件路径匹配语法,参考<https://golang.org/pkg/path/filepath/#Match>

```dockerfile
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

示例2

`<dest>`是绝对路径或是相对路径（相对于`WORKDIR`）

```dockerfile
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

支持权限设置

```dockerfile
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```

## COPY

语法

- `COPY [--chown=<user>:<group>] <src>... <dest>`
- `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]` 该格式支持包含空格情况

与`ADD`基本一样，但是不会进行解压

## ENTRYPOINT

语法

- `ENTRYPOINT ["executable", "param1", "param2"]` （exec格式，首选）
- `ENTRYPOINT command param1 param2` （shell格式）

`ENTRYPOINT`允许您配置将作为可执行文件运行的容器。

## VOLUME

语法

- `VOLUME ["/data"]`

`VOLUME`指令使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用。我们知道容器使用的是AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。当容器中的应用有持久化数据的需求时可以在Dockerfile中使用该指令。

该指令参数可以是一个JSON数组`VOLUME ["/var/log/"]`，或者是多参数形式的字符串`VOLUME /var/log`或者是`VOLUME /var/log /var/db`。

## 示例

```dockerfile
# Nginx
#
# VERSION               0.0.1

FROM      ubuntu
LABEL Description="This image is used to start the foobar executable" Vendor="ACME Products" Version="1.0"
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
```

```dockerfile
# Firefox over VNC
#
# VERSION               0.3

FROM ubuntu

# Install vnc, xvfb in order to create a 'fake' display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir ~/.vnc
# Setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the trick)
RUN bash -c 'echo "firefox" >> /.bashrc'

EXPOSE 5900
CMD    ["x11vnc", "-forever", "-usepw", "-create"]
```

### 多镜像示例

```dockerfile
FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f

FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4

# You'll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
# /oink.
```
