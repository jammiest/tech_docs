# 用户权限管理

## 查看

> 语法`SHOW GRANTS [FOR user]`

demo

```mysql
SHOW GRANTS FOR 'jeffrey'@'localhost';
```

## 创建用户

```mysql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';
```

## 修改权限

demo

```mysql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';
GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
GRANT SELECT ON db2.invoice TO 'jeffrey'@'localhost';
ALTER USER 'jeffrey'@'localhost' WITH MAX_QUERIES_PER_HOUR 90;
-- ---------------------或-----------------------
CREATE USER 'admin'@'192.168.%' IDENTIFIED BY 'Admin123!';
GRANT SELECT,SHOW VIEW ON *.* TO 'admin'@'192.168.%';
FLUSH PRIVILEGES;
```

## 删除用户

删除用户

```mysql
DROP USER 'jeffrey'@'localhost';
FLUSH PRIVILEGES;
```

!> 删除匿名用户

```mysql
DROP USER ''@'localhost';
FLUSH PRIVILEGES;
```
