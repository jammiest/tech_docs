## 初始环境说明
4台纯净的CentOs Minimal虚拟机环境  
VM0(作为Master)：192.168.0.254  

VM1（作为SLAVE）:192.168.0.241  
VM2（作为SLAVE）:192.168.0.242  
VM3（作为SLAVE）:192.168.0.243  

## MYSQL SERVER 安装

- 借助Xshell等SSH工具批量命令安装和配置mysql-community-server，并启动mysql [ 推荐yum包安装方式，详情参考文章最下方的参考文档 ]

!>  本教程基于MYSQL5.7版本，其他版本可能略有不同，详情参考官方相关文档

1、安装MySQL官方的RPM文件[MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/)并且安装

> CentOS是EL7架构的，因此选择'Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package'，通过以下命令查看
```bash
uname -a
```  

```bash
# 安装必要软件
yum install vim wget yum-config-manager
# 通过wget方法下载到CentOS当前目录下
wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
# 安装RPM package
sudo yum localinstall mysql80-community-release-el7-1.noarch.rpm
```
2、安装完成RPM包，可以查看默认可安装的MYSQL版本，通过执行命令

```bash
yum repolist enabled | grep "mysql.*-community.*"
``` 

3、查看所有支持安装的MYSQL版本，通过执行命令`yum repolist all | grep mysql`  
4、切换MYSQL server默认版本为5.7  

```bash
yum-config-manager --disable mysql80-community
yum-config-manager --enable mysql57-community
```

5、安装并且启动MYSQL

```bash
yum install mysql-community-server
service mysqld status
```

6、MYSQL在第一次安装完成会为root设置临时随机密码，通过命令查看

```bash
grep 'temporary password' /var/log/mysqld.log
```

7、初次登录MYSQL并且修改密码

```bash
mysql -uroot -p 
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Jiang123!';
```

- 关闭和禁用防火墙[为便于测试]

```bash
systemctl disable firewalld
systemctl stop firewalld
```

## 主从设置

1、进入VM0，新增同步账号以及设置相应权限
```sql
-- 创建专门用于主从同步的MYSQL账号，用户名repl，密码password，允许访问的IP地址192.168.0.%（%为通配符）
CREATE USER 'repl'@'192.168.0.%' IDENTIFIED BY 'Repl123!';
-- 给repl账号授权REPLICATION SLAVE权限
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.0.%';
FLUSH PRIVILEGES;
```

2、分别为主从MYSQL配置文件[/etc/my.cnf]进行主从相关配置
> master服务器

```ini
# server-id 服务器标识ID，每个服务器的server-id都不一样，考虑到以后会涉及多主多从的架构，建议master设置为个位数
server-id=1
log-bin=mysql-bin
```

> slave服务器

```ini
# server-id 服务器标识ID，考虑到以后会涉及多主多从的架构，slave为非个位数ID
server-id=244
```

> 然后重启所有服务器（VM0,VM1,VM3,VM3）

3、master服务器（VM0）

```mysql
-- 刷新所有表以及阻止写操作
FLUSH TABLES WITH READ LOCK;

-- 查看log文件名和位置
SHOW MASTER STATUS;
```

4、进入slave的MYSQL server命令行，通过执行`mysql -u root -p`

```sql
-- 关闭和重置主从同步，首次配置可省略这两步骤
STOP SLAVE;
RESET SLAVE;
-- MASTER_PASSWORD的值通过在master mysql执行命令`show master status;`查询
-- MASTER_HOST可以为域名或IP，建议设置为域名，便于到/etc/hosts中设置，和后期的调整
CHANGE MASTER TO MASTER_HOST='master1', MASTER_USER='repl', MASTER_PORT=3306, MASTER_PASSWORD='Repl123!', MASTER_LOG_FILE='master-bin.000050', master_log_pos=0;
-- 启动主从同步
start slave;
```

查主从同步状态

```mysql
-- VM0（master服务器查看）
show master status;

-- VM1,VM2,VM3（slave服务器查看）
show slave status;

```


5、创建主库的远程管理账号以及权限 VM0

```sql
CREATE USER 'admin'@'192.168.%' IDENTIFIED BY 'Admin123!';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, INDEX, ALTER, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, CREATE VIEW, SHOW VIEW, CREATE TABLESPACE ON *.* TO 'admin'@'192.168.%';
FLUSH PRIVILEGES;
```

6、创建从库的远程管理账号以及权限 VM1 VM2 VM3  
**为兼容更多框架MYSQL读写分离架构的语法特点，故采取保持主从库账号以及密码一致性，仅对权限不同区分**

```sql
CREATE USER 'admin'@'192.168.%' IDENTIFIED BY 'Admin123!';
GRANT SELECT,SHOW VIEW,REPLICATION CLIENT ON *.* TO 'admin'@'192.168.%';
FLUSH PRIVILEGES;
```

## 进阶

### 主从延迟时间设置

通过对VM1,VM3,VM3其中一台（SLAVE）数据库设置主从延迟时长（单位秒），语法如下所示

```mysql
CHANGE MASTER TO MASTER_DELAY = N;
```

可实现一下目的：

- DBA能够在主数据库出错是时，进行数据库的容灾回归
- 数据库高I/O情况下的主从延迟问题环境模拟
- 加载备份库情况下对比如一个星期前的延迟SLAVE的数据

?> `START SLAVE` 和 `STOP SLAVE` 立即生效和忽略任何延迟，`RESET SLAVE` 重置延迟时长为0

## 官方参考文档

> [mysql yum包安装](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html)  
> [权限管理](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html)  
> [主从复制](https://dev.mysql.com/doc/refman/5.7/en/replication.html)