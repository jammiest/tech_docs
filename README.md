#

## 文档在线预览

 - <http://tech.03pl.com/#/>
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


## 目录

- [PHP](https://github.com/jammiest/tech_docs/blob/master/person/php/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/php/)
  - [PHP7.3.5编译](https://github.com/jammiest/tech_docs/blob/master/person/php/PHP7.3.5编译)
  - [PHP杂项](https://github.com/jammiest/tech_docs/blob/master/person/php/PHP杂项)
  - [Laravel.消息队列](https://github.com/jammiest/tech_docs/blob/master/person/php/Laravel.消息队列)
  - [RabbitMQ](https://github.com/jammiest/tech_docs/blob/master/person/php/RabbitMQ)
- [数据库](https://github.com/jammiest/tech_docs/blob/master/person/数据库/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/数据库/README)
  - [MYSQL主从架构搭建](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL主从架构搭建)
  - [MYSQL用户权限管理](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL用户权限管理-grant)
  - [MYSQL存储例程语法（数据库编程）](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL存储例程语法)
  - [MYSQL事务](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL事务)
  - [MYSQL错误码和信息](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL错误码和信息)  
- [消息队列](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/README)
  - [RabbitMQ基础概念(一)](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/RabbitMQ基础概念\(一\))
  - [RabbitMQ高级特性(二)](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/RabbitMQ高级特性\(二\))
  - [RabbitMQ生产端保证消息100投递成功](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/RabbitMQ生产端保证消息100投递成功)
  
- [文件服务](https://github.com/jammiest/tech_docs/blob/master/person/文件服务/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Redis/README)
  
- [Redis](https://github.com/jammiest/tech_docs/blob/master/person/Redis/)  
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Redis/README)
  - [Redis集群初级教程](https://github.com/jammiest/tech_docs/blob/master/person/Redis/Redis集群初级教程)
  - [Redis常用指令](https://github.com/jammiest/tech_docs/blob/master/person/Redis/Redis常用指令)
- [nginx](https://github.com/jammiest/tech_docs/blob/master/person/nginx/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/nginx/README)
  - [Win本地环境搭建](https://github.com/jammiest/tech_docs/blob/master/person/nginx/Win本地环境搭建)
  - [虚拟机本地环境搭建](https://github.com/jammiest/tech_docs/blob/master/person/nginx/虚拟机本地环境搭建)
  - [Nginx变量](https://github.com/jammiest/tech_docs/blob/master/person/nginx/Nginx变量)
  - [location](https://github.com/jammiest/tech_docs/blob/master/person/nginx/location)
  - [状态码](https://github.com/jammiest/tech_docs/blob/master/person/nginx/状态码)
  - [条件判断及运算符](https://github.com/jammiest/tech_docs/blob/master/person/nginx/条件判断及运算符)
  - [Nginx负载均衡](https://github.com/jammiest/tech_docs/blob/master/person/nginx/Nginx负载均衡)
  
- [GIT](https://github.com/jammiest/tech_docs/blob/master/person/GIT/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/GIT/README)
  - [日志输出格式化](https://github.com/jammiest/tech_docs/blob/master/person/GIT/日志输出格式化)
  - [update-index](https://github.com/jammiest/tech_docs/blob/master/person/GIT/update-index)
  - [PowerShell Git配置](https://github.com/jammiest/tech_docs/blob/master/person/GIT/PowerShell)
  - [Commitizen](https://github.com/jammiest/tech_docs/blob/master/person/GIT/Commitizen)
  - [钩子](https://github.com/jammiest/tech_docs/blob/master/person/GIT/钩子)
  - [Repository 仓库](https://github.com/jammiest/tech_docs/blob/master/person/GIT/Repository（仓库）)
- [算法](https://github.com/jammiest/tech_docs/blob/master/person/Algorithm/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Algorithm/README)
  - [散列表](https://github.com/jammiest/tech_docs/blob/master/person/Algorithm/Hash)
- [Python](https://github.com/jammiest/tech_docs/blob/master/person/Python/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Python/README)
  - [RabbitMQ](https://github.com/jammiest/tech_docs/blob/master/person/Python/RabbitMQ)

- [正则](https://github.com/jammiest/tech_docs/blob/master/person/正则/)
  - [正则语法](https://github.com/jammiest/tech_docs/blob/master/person/正则/正则语法)
  - [PHP代码匹配](https://github.com/jammiest/tech_docs/blob/master/person/正则/PHP代码查询匹配)

- [Shell](https://github.com/jammiest/tech_docs/blob/master/person/Shell/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Shell/README)
  - [firewalld](https://github.com/jammiest/tech_docs/blob/master/person/Shell/防火墙)
  - [Grep](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Grep)
  - [Sed](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Sed)
  - [Awk](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Awk)
  - [vim常用命令](https://github.com/jammiest/tech_docs/blob/master/person/Shell/vim常用命令)
  - [History](https://github.com/jammiest/tech_docs/blob/master/person/Shell/History)
  - [KiLL](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Kill&Killall)
  - [Curl](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Curl)
  - [Shell脚本趣玩算法](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell脚本趣玩算法)
  - [Shell运算及统计](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell运算及统计)
  - [Shell编程大全](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell编程大全)
  - [BashShell配置](https://github.com/jammiest/tech_docs/blob/master/person/Shell/BashShell配置)
  - [Shell脚本配色](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell脚本配色)
  - [PowerShell](https://github.com/jammiest/tech_docs/blob/master/person/Shell/PowerShell)

- [OS](https://github.com/jammiest/tech_docs/blob/master/person/OS/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/OS/Linux/README)
  - [系统时间管理](https://github.com/jammiest/tech_docs/blob/master/person/OS/Linux/系统时间管理)
  - [crontab](https://github.com/jammiest/tech_docs/blob/master/person/Linux/OS/crontab)
  - [Win10常见功能开启](https://github.com/jammiest/tech_docs/blob/master/person/Win/Win10常见功能开启)
  - [Win10开机启动脚本](https://github.com/jammiest/tech_docs/blob/master/person/Win/Win10开机启动脚本)

- [Docker](https://github.com/jammiest/tech_docs/blob/master/person/Docker/)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Docker/README)
  - [基本指令预览](https://github.com/jammiest/tech_docs/blob/master/person/Docker/基本指令预览)
  - [Dockerfile语法](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Dockerfile语法)
  - [Docsify容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Docsify容器构建)
  - [PHP容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/PHP容器构建)
  - [Nginx容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Nginx容器构建)
  - [Nginx容器支持FastDFS模块](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Nginx容器支持FastDFS模块)
  - [FastDFS容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/FastDFS容器构建)
  - [VMware虚拟机扩容](https://github.com/jammiest/tech_docs/blob/master/person/Docker/VMware/VMware扩容)

- [测试运维](https://github.com/jammiest/tech_docs/blob/master/person/测试运维/)

- [架构](https://github.com/jammiest/tech_docs/blob/master/person/架构/)

- [ELK](https://github.com/jammiest/tech_docs/blob/master/person/ELK/)
  
  