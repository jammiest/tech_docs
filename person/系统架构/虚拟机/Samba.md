# Samba

## 安装samba

```bash
yum install samba samba-client
```

## 开机自启动

```bash
chkconfig smb on
chkconfig nmb on
```

## 启动服务

```bash
service smb start
service nmb start
```

## 添加组

```bash
groupadd samba
useradd web
usermod -G samba web
smbpasswd -a web
```

## 关闭防火墙

```bash
systemctl stop firewalld
```

## 改变当前目录所属的用户和组

```bash
chgrp samba /var/samba
chown -R web /var/samba
```

## 配置文件

```bash
vim /etc/samba/smb.conf
```

## Windows访问

假设samba服务器的ip地址是 **192.168.1.99**

> 在浏览器中输入`\\192.168.1.99`,根据提示输入用户凭证信息即可
