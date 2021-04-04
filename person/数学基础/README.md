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


## Github目录导航

- [PHP](https://github.com/jammiest/tech_docs/blob/master/person/PHP/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/PHP/README.md)
  - [PHP7.3.5编译](https://github.com/jammiest/tech_docs/blob/master/person/PHP/PHP7.3.5编译.md)
  - [PHP杂项](https://github.com/jammiest/tech_docs/blob/master/person/PHP/PHP杂项.md)
  - [Laravel.消息队列](https://github.com/jammiest/tech_docs/blob/master/person/PHP/Laravel.消息队列.md)
  - [RabbitMQ](https://github.com/jammiest/tech_docs/blob/master/person/PHP/RabbitMQ.md)
  
- [数据库](https://github.com/jammiest/tech_docs/blob/master/person/数据库/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/数据库/README.md)
  - [MYSQL主从架构搭建](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL主从架构搭建.md)
  - [MYSQL用户权限管理](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL用户权限管理-grant.md)
  - [MYSQL存储例程语法（数据库编程）](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL存储例程语法.md)
  - [MYSQL事务](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL事务.md)
  - [MYSQL错误码和信息](https://github.com/jammiest/tech_docs/blob/master/person/数据库/MYSQL错误码和信息.md)
  
- [消息队列](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/README.md)
  - [RabbitMQ基础概念(一)](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/RabbitMQ基础概念\(一\).md)
  - [RabbitMQ高级特性(二)](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/RabbitMQ高级特性\(二\).md)
  - [RabbitMQ生产端保证消息100投递成功](https://github.com/jammiest/tech_docs/blob/master/person/消息队列/RabbitMQ生产端保证消息100投递成功.md)
  
- [文件服务](https://github.com/jammiest/tech_docs/blob/master/person/文件服务/README.md)
  
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Redis/README.md)
  
- [Redis](https://github.com/jammiest/tech_docs/blob/master/person/Redis/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Redis/README.md)
  - [Redis集群初级教程](https://github.com/jammiest/tech_docs/blob/master/person/Redis/Redis集群初级教程.md)
  - [Redis常用指令](https://github.com/jammiest/tech_docs/blob/master/person/Redis/Redis常用指令.md)
  
- [nginx](https://github.com/jammiest/tech_docs/blob/master/person/nginx/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/nginx/README.md)
  - [Win本地环境搭建](https://github.com/jammiest/tech_docs/blob/master/person/nginx/Win本地环境搭建.md)
  - [虚拟机本地环境搭建](https://github.com/jammiest/tech_docs/blob/master/person/nginx/虚拟机本地环境搭建.md)
  - [Nginx变量](https://github.com/jammiest/tech_docs/blob/master/person/nginx/Nginx变量.md)
  - [location](https://github.com/jammiest/tech_docs/blob/master/person/nginx/location.md)
  - [状态码](https://github.com/jammiest/tech_docs/blob/master/person/nginx/状态码.md)
  - [条件判断及运算符](https://github.com/jammiest/tech_docs/blob/master/person/nginx/条件判断及运算符.md)
  - [Nginx负载均衡](https://github.com/jammiest/tech_docs/blob/master/person/nginx/Nginx负载均衡.md)
  
- [GIT](https://github.com/jammiest/tech_docs/blob/master/person/GIT/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/GIT/README.md)
  - [日志输出格式化](https://github.com/jammiest/tech_docs/blob/master/person/GIT/日志输出格式化.md)
  - [update-index](https://github.com/jammiest/tech_docs/blob/master/person/GIT/update-index.md)
  - [PowerShell Git配置](https://github.com/jammiest/tech_docs/blob/master/person/GIT/PowerShell.md)
  - [Commitizen](https://github.com/jammiest/tech_docs/blob/master/person/GIT/Commitizen.md)
  - [钩子](https://github.com/jammiest/tech_docs/blob/master/person/GIT/钩子.md)
  - [Repository 仓库](https://github.com/jammiest/tech_docs/blob/master/person/GIT/Repository（仓库）.md)
  
- [算法](https://github.com/jammiest/tech_docs/blob/master/person/Algorithm/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Algorithm/README.md)
  - [散列表](https://github.com/jammiest/tech_docs/blob/master/person/Algorithm/Hash.md)
  
- [Python](https://github.com/jammiest/tech_docs/blob/master/person/Python/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Python/README.md)
  - [RabbitMQ](https://github.com/jammiest/tech_docs/blob/master/person/Python/RabbitMQ.md)

- [正则](https://github.com/jammiest/tech_docs/blob/master/person/正则/README.md)
  - [正则语法](https://github.com/jammiest/tech_docs/blob/master/person/正则/正则语法.md)
  - [PHP代码匹配](https://github.com/jammiest/tech_docs/blob/master/person/正则/PHP代码查询匹配.md)

- [Shell](https://github.com/jammiest/tech_docs/blob/master/person/Shell/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Shell/README.md)
  - [firewalld](https://github.com/jammiest/tech_docs/blob/master/person/Shell/防火墙.md)
  - [Grep](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Grep.md)
  - [Sed](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Sed.md)
  - [Awk](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Awk.md)
  - [vim常用命令](https://github.com/jammiest/tech_docs/blob/master/person/Shell/vim常用命令.md)
  - [History](https://github.com/jammiest/tech_docs/blob/master/person/Shell/History.md)
  - [KiLL](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Kill&Killall.md)
  - [Curl](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Curl.md)
  - [Shell脚本趣玩算法](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell脚本趣玩算法.md)
  - [Shell运算及统计](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell运算及统计.md)
  - [Shell编程大全](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell编程大全.md)
  - [BashShell配置](https://github.com/jammiest/tech_docs/blob/master/person/Shell/BashShell配置.md)
  - [Shell脚本配色](https://github.com/jammiest/tech_docs/blob/master/person/Shell/Shell脚本配色.md)
  - [PowerShell](https://github.com/jammiest/tech_docs/blob/master/person/Shell/PowerShell.md)

- [OS](https://github.com/jammiest/tech_docs/blob/master/person/OS/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/OS/Linux/README.md)
  - [系统时间管理](https://github.com/jammiest/tech_docs/blob/master/person/OS/Linux/系统时间管理.md)
  - [crontab](https://github.com/jammiest/tech_docs/blob/master/person/Linux/OS/crontab.md)
  - [Win10常见功能开启](https://github.com/jammiest/tech_docs/blob/master/person/Win/Win10常见功能开启.md)
  - [Win10开机启动脚本](https://github.com/jammiest/tech_docs/blob/master/person/Win/Win10开机启动脚本.md)

- [Docker](https://github.com/jammiest/tech_docs/blob/master/person/Docker/README.md)
  - [README](https://github.com/jammiest/tech_docs/blob/master/person/Docker/README.md)
  - [基本指令预览](https://github.com/jammiest/tech_docs/blob/master/person/Docker/基本指令预览.md)
  - [Dockerfile语法](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Dockerfile语法.md)
  - [Composefile语法](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Composefile语法.md)
  - [Docsify容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Docsify容器构建.md)
  - [PHP容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/PHP容器构建.md)
  - [Nginx容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Nginx容器构建.md)
  - [Nginx容器支持FastDFS模块](https://github.com/jammiest/tech_docs/blob/master/person/Docker/Nginx容器支持FastDFS模块.md)
  - [FastDFS容器构建](https://github.com/jammiest/tech_docs/blob/master/person/Docker/FastDFS容器构建.md)
  - [VMware虚拟机扩容](https://github.com/jammiest/tech_docs/blob/master/person/Docker/VMware/VMware扩容.md)

- [测试运维](https://github.com/jammiest/tech_docs/blob/master/person/测试运维/README.md)

- [架构](https://github.com/jammiest/tech_docs/blob/master/person/架构/README.md)

- [ELK](https://github.com/jammiest/tech_docs/blob/master/person/ELK/README.md)
  
  
  
  [Markdown语法](https://github.com/jammiest/tech_docs/blob/master/person/ELK/markdown.md)
  
  

