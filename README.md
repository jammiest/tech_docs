#

## 文档在线预览

 - <https://jammiest.github.io/tech_docs/#/>


## 文档系统本地部署

参考：<https://docsify.js.org/#/zh-cn/quickstart?id=初始化项目>

```shell
# 进入目录

# 安装工具
npm i docsify-cli -g

# 开启服务
docsify serve
```

本地访问 <http://localhost:3000>

## 服务器部署

参考：<https://docsify.js.org/#/zh-cn/deploy>

nginx 的配置

```nginx
server {
  listen 80;
  server_name  your.domain.com;

  location / {
    alias /path/to/dir/of/docs;
    index index.html;
  }
}
```


## 目录导航

- [PHP](person/PHP/README.md)
  - [README](person/PHP/README.md)
  - [PHP7.3.5编译](person/PHP/PHP7.3.5编译.md)
  - [PHP杂项](person/PHP/PHP杂项.md)
  - [Laravel.消息队列](person/PHP/Laravel.消息队列.md)
  - [RabbitMQ](person/PHP/RabbitMQ.md)
  
- [数据库](person/数据库/README.md)
  - [README](person/数据库/README.md)
  - [MYSQL主从架构搭建](person/数据库/MYSQL主从架构搭建.md)
  - [MYSQL用户权限管理](person/数据库/MYSQL用户权限管理-grant.md)
  - [MYSQL存储例程语法（数据库编程）](person/数据库/MYSQL存储例程语法.md)
  - [MYSQL事务](person/数据库/MYSQL事务.md)
  - [MYSQL错误码和信息](person/数据库/MYSQL错误码和信息.md)
  
- [中间件](person/中间件/README.md)
  - [README](person/中间件/README.md)
  - [RabbitMQ基础概念(一)](person/中间件/RabbitMQ基础概念\(一\).md)
  - [RabbitMQ高级特性(二)](person/中间件/RabbitMQ高级特性\(二\).md)
  - [RabbitMQ生产端保证消息100投递成功](person/中间件/RabbitMQ生产端保证消息100投递成功.md)
  
- [文件服务](person/文件服务/README.md)
  
  - [README](person/Redis/README.md)
  
- [Redis](person/Redis/README.md)
  - [README](person/Redis/README.md)
  - [Redis集群初级教程](person/Redis/Redis集群初级教程.md)
  - [Redis常用指令](person/Redis/Redis常用指令.md)
  
- [nginx](person/nginx/README.md)
  - [README](person/nginx/README.md)
  - [Win本地环境搭建](person/nginx/Win本地环境搭建.md)
  - [虚拟机本地环境搭建](person/nginx/虚拟机本地环境搭建.md)
  - [Nginx变量](person/nginx/Nginx变量.md)
  - [location](person/nginx/location.md)
  - [状态码](person/nginx/状态码.md)
  - [条件判断及运算符](person/nginx/条件判断及运算符.md)
  - [Nginx负载均衡](person/nginx/Nginx负载均衡.md)
  
- [GIT](person/GIT/README.md)
  - [README](person/GIT/README.md)
  - [日志输出格式化](person/GIT/日志输出格式化.md)
  - [update-index](person/GIT/update-index.md)
  - [PowerShell Git配置](person/GIT/PowerShell.md)
  - [Commitizen](person/GIT/Commitizen.md)
  - [钩子](person/GIT/钩子.md)
  - [Repository 仓库](person/GIT/Repository（仓库）.md)
  
- [算法](person/Algorithm/README.md)
  - [README](person/Algorithm/README.md)
  - [散列表](person/Algorithm/Hash.md)
  
- [Python](person/Python/README.md)
  - [README](person/Python/README.md)
  - [RabbitMQ](person/Python/RabbitMQ.md)

- [正则](person/正则/README.md)
  - [正则语法](person/正则/正则语法.md)
  - [PHP代码匹配](person/正则/PHP代码查询匹配.md)

- [Shell](person/Shell/README.md)
  - [README](person/Shell/README.md)
  - [firewalld](person/Shell/防火墙.md)
  - [Grep](person/Shell/Grep.md)
  - [Sed](person/Shell/Sed.md)
  - [Awk](person/Shell/Awk.md)
  - [vim常用命令](person/Shell/vim常用命令.md)
  - [History](person/Shell/History.md)
  - [KiLL](person/Shell/Kill&Killall.md)
  - [Curl](person/Shell/Curl.md)
  - [Shell脚本趣玩算法](person/Shell/Shell脚本趣玩算法.md)
  - [Shell运算及统计](person/Shell/Shell运算及统计.md)
  - [Shell编程大全](person/Shell/Shell编程大全.md)
  - [BashShell配置](person/Shell/BashShell配置.md)
  - [Shell脚本配色](person/Shell/Shell脚本配色.md)
  - [PowerShell](person/Shell/PowerShell.md)

- [OS](person/OS/README.md)
  - [README](person/OS/Linux/README.md)
  - [系统时间管理](person/OS/Linux/系统时间管理.md)
  - [crontab](person/Linux/OS/crontab.md)
  - [Win10常见功能开启](person/Win/Win10常见功能开启.md)
  - [Win10开机启动脚本](person/Win/Win10开机启动脚本.md)

- [Docker](person/Docker/README.md)
  - [README](person/Docker/README.md)
  - [基本指令预览](person/Docker/基本指令预览.md)
  - [Dockerfile语法](person/Docker/Dockerfile语法.md)
  - [Composefile语法](person/Docker/Composefile语法.md)
  - [Docsify容器构建](person/Docker/Docsify容器构建.md)
  - [PHP容器构建](person/Docker/PHP容器构建.md)
  - [Nginx容器构建](person/Docker/Nginx容器构建.md)
  - [Nginx容器支持FastDFS模块](person/Docker/Nginx容器支持FastDFS模块.md)
  - [FastDFS容器构建](person/Docker/FastDFS容器构建.md)
  - [VMware虚拟机扩容](person/Docker/VMware/VMware扩容.md)

- [测试运维](person/测试运维/README.md)

- [架构](person/架构/README.md)

- [ELK](person/ELK/README.md)
  
  
  
  [Markdown语法](person/ELK/markdown.md)
  
  

