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

- [PHP](/person/php/)
  - [README](/person/php/)
  - [PHP7.3.5编译](/person/php/PHP7.3.5编译)
  - [PHP杂项](/person/php/PHP杂项)
  - [Laravel.消息队列](/person/php/Laravel.消息队列)
  - [RabbitMQ](/person/php/RabbitMQ)
- [数据库](/person/数据库/)
  - [README](/person/数据库/README)
  - [MYSQL主从架构搭建](/person/数据库/MYSQL主从架构搭建)
  - [MYSQL用户权限管理](/person/数据库/MYSQL用户权限管理-grant)
  - [MYSQL存储例程语法（数据库编程）](/person/数据库/MYSQL存储例程语法)
  - [MYSQL事务](/person/数据库/MYSQL事务)
  - [MYSQL错误码和信息](/person/数据库/MYSQL错误码和信息)  
- [消息队列](/person/消息队列/)
  - [README](/person/消息队列/README)
  - [RabbitMQ基础概念(一)](/person/消息队列/RabbitMQ基础概念\(一\))
  - [RabbitMQ高级特性(二)](/person/消息队列/RabbitMQ高级特性\(二\))
  - [RabbitMQ生产端保证消息100投递成功](/person/消息队列/RabbitMQ生产端保证消息100投递成功)
  
- [文件服务](/person/文件服务/)
  - [README](/person/Redis/README)
  
- [Redis](/person/Redis/)  
  - [README](/person/Redis/README)
  - [Redis集群初级教程](/person/Redis/Redis集群初级教程)
  - [Redis常用指令](/person/Redis/Redis常用指令)
- [nginx](/person/nginx/)
  - [README](/person/nginx/README)
  - [Win本地环境搭建](/person/nginx/Win本地环境搭建)
  - [虚拟机本地环境搭建](/person/nginx/虚拟机本地环境搭建)
  - [Nginx变量](/person/nginx/Nginx变量)
  - [location](/person/nginx/location)
  - [状态码](/person/nginx/状态码)
  - [条件判断及运算符](/person/nginx/条件判断及运算符)
  - [Nginx负载均衡](/person/nginx/Nginx负载均衡)
  
- [GIT](/person/GIT/)
  - [README](/person/GIT/README)
  - [日志输出格式化](/person/GIT/日志输出格式化)
  - [update-index](/person/GIT/update-index)
  - [PowerShell Git配置](/person/GIT/PowerShell)
  - [Commitizen](/person/GIT/Commitizen)
  - [钩子](/person/GIT/钩子)
  - [Repository 仓库](/person/GIT/Repository（仓库）)
- [算法](/person/Algorithm/)
  - [README](/person/Algorithm/README)
  - [散列表](/person/Algorithm/Hash)
- [Python](/person/Python/)
  - [README](/person/Python/README)
  - [RabbitMQ](/person/Python/RabbitMQ)

- [正则](/person/正则/)
  - [正则语法](/person/正则/正则语法)
  - [PHP代码匹配](/person/正则/PHP代码查询匹配)

- [Shell](/person/Shell/)
  - [README](/person/Shell/README)
  - [firewalld](/person/Shell/防火墙)
  - [Grep](/person/Shell/Grep)
  - [Sed](/person/Shell/Sed)
  - [Awk](/person/Shell/Awk)
  - [vim常用命令](/person/Shell/vim常用命令)
  - [History](/person/Shell/History)
  - [KiLL](/person/Shell/Kill&Killall)
  - [Curl](/person/Shell/Curl)
  - [Shell脚本趣玩算法](/person/Shell/Shell脚本趣玩算法)
  - [Shell运算及统计](/person/Shell/Shell运算及统计)
  - [Shell编程大全](/person/Shell/Shell编程大全)
  - [BashShell配置](/person/Shell/BashShell配置)
  - [Shell脚本配色](/person/Shell/Shell脚本配色)
  - [PowerShell](/person/Shell/PowerShell)

- [OS](/person/OS/)
  - [README](/person/Linux/README)
  - [系统时间管理](/person/Linux/系统时间管理)
  - [crontab](/person/Linux/crontab)
  - [Win10常见功能开启](/person/Win/Win10常见功能开启)
  - [Win10开机启动脚本](/person/Win/Win10开机启动脚本)

- [Docker](/person/Docker/)
  - [README](/person/Docker/README)
  - [基本指令预览](/person/Docker/基本指令预览)
  - [Dockerfile语法](/person/Docker/Dockerfile语法)
  - [Docsify容器构建](/person/Docker/Docsify容器构建)
  - [PHP容器构建](/person/Docker/PHP容器构建)
  - [Nginx容器构建](/person/Docker/Nginx容器构建)
  - [Nginx容器支持FastDFS模块](/person/Docker/Nginx容器支持FastDFS模块)
  - [FastDFS容器构建](/person/Docker/FastDFS容器构建)
  - [VMware虚拟机扩容](/person/Docker/VMware/VMware扩容)

- [测试运维](/person/测试运维/)

- [架构](/person/架构/)

- [ELK](/person/ELK/)
  
  