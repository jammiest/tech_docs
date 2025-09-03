# MySQL 容器化部署完全指南

## 基础部署

### 快速启动 MySQL 容器
```bash
# 最新版本 MySQL
docker run -d \
  --name mysql-server \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  -p 3306:3306 \
  mysql:latest

# 指定版本 (如 8.0)
docker run -d \
  --name mysql-8.0 \
  -e MYSQL_ROOT_PASSWORD=secure_password \
  -p 3307:3306 \
  mysql:8.0
```

### 核心环境变量
| 变量名 | 描述 | 示例值 |
|--------|------|--------|
| `MYSQL_ROOT_PASSWORD` | root 用户密码 | my-secret-pw |
| `MYSQL_DATABASE` | 自动创建的数据库 | mydb |
| `MYSQL_USER` | 自动创建的用户 | myuser |
| `MYSQL_PASSWORD` | 自动用户的密码 | userpass |
| `MYSQL_ALLOW_EMPTY_PASSWORD` | 允许空密码 | yes |
| `MYSQL_RANDOM_ROOT_PASSWORD` | 随机生成 root 密码 | yes |

## 数据持久化配置

### 使用数据卷持久化
```bash
# 创建专用数据卷
docker volume create mysql_data

# 运行带持久化的 MySQL
docker run -d \
  --name mysql-persistent \
  -v mysql_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0
```

### 绑定挂载配置
```bash
# 使用主机目录存储数据
mkdir -p /opt/mysql/{data,conf,logs}

# 运行 MySQL 并挂载配置目录
docker run -d \
  --name mysql-custom \
  -v /opt/mysql/data:/var/lib/mysql \
  -v /opt/mysql/conf:/etc/mysql/conf.d \
  -v /opt/mysql/logs:/var/log/mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0
```

## 配置文件定制

### 自定义 my.cnf 配置
```bash
# 创建自定义配置文件目录
mkdir -p /opt/mysql/conf.d

# 添加配置文件
cat > /opt/mysql/conf.d/my-custom.cnf <<EOF
[mysqld]
max_connections = 500
innodb_buffer_pool_size = 1G
query_cache_size = 0
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[client]
default-character-set = utf8mb4
EOF

# 运行带自定义配置的 MySQL
docker run -d \
  --name mysql-custom-config \
  -v /opt/mysql/conf.d:/etc/mysql/conf.d \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0
```

### 常用性能参数
```ini
[mysqld]
# 内存配置
innodb_buffer_pool_size = 2G
innodb_log_file_size = 256M
key_buffer_size = 128M

# 连接配置
max_connections = 300
thread_cache_size = 50
table_open_cache = 2000

# 日志配置
slow_query_log = 1
long_query_time = 2
log_queries_not_using_indexes = 1

# 其他优化
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
```

## 安全配置

### 安全启动选项
```bash
# 使用随机 root 密码
docker run -d \
  --name mysql-secure \
  -e MYSQL_RANDOM_ROOT_PASSWORD=yes \
  mysql:8.0

# 自动创建用户和数据库
docker run -d \
  --name mysql-app \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=appdb \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=userpass \
  mysql:8.0
```

### 生产环境安全建议
1. **避免使用 root 用户**：为应用创建专用用户
2. **最小权限原则**：只授予必要权限
3. **启用 SSL**：配置 SSL 加密连接
4. **定期备份**：设置自动化备份策略
5. **日志审计**：启用查询日志和错误日志

## 网络与连接

### 网络配置
```bash
# 创建专用网络
docker network create mysql-network

# 在专用网络中运行 MySQL
docker run -d \
  --name mysql-net \
  --network mysql-network \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0

# 连接到此网络的应用程序
docker run -d \
  --name myapp \
  --network mysql-network \
  -e DB_HOST=mysql-net \
  -e DB_PASSWORD=password \
  myapp:latest
```

### 连接测试
```bash
# 使用 MySQL 客户端连接
docker exec -it mysql-server mysql -uroot -p

# 从外部主机连接
mysql -h 127.0.0.1 -P 3306 -u root -p

# 使用临时客户端容器连接
docker run -it --rm \
  --network mysql-network \
  mysql:8.0 \
  mysql -hmysql-net -uroot -ppassword
```

## 备份与恢复

### 数据库备份
```bash
# 使用 mysqldump 备份
docker exec mysql-server \
  mysqldump -uroot -ppassword --all-databases > backup.sql

# 备份特定数据库
docker exec mysql-server \
  mysqldump -uroot -ppassword mydb > mydb_backup.sql

# 定时备份脚本
docker run -d \
  --name mysql-backup \
  --volumes-from mysql-server \
  -v /backups:/backups \
  -e MYSQL_ROOT_PASSWORD=password \
  alpine \
  sh -c 'while true; do \
    docker exec mysql-server \
      mysqldump -uroot -ppassword --all-databases > /backups/mysql-$(date +%Y%m%d%H%M%S).sql; \
    sleep 86400; done'
```

### 数据恢复
```bash
# 恢复完整备份
cat backup.sql | docker exec -i mysql-server mysql -uroot -ppassword

# 恢复单个数据库
docker exec -i mysql-server mysql -uroot -ppassword mydb < mydb_backup.sql
```

## 性能优化

### 资源限制
```bash
# 限制容器资源
docker run -d \
  --name mysql-optimized \
  --cpus=2 \
  --memory=4g \
  --memory-swap=4g \
  --ulimit nofile=65536:65536 \
  -v mysql_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0 \
  --innodb_buffer_pool_size=2G \
  --innodb_log_file_size=512M
```

### 生产环境推荐参数
```bash
docker run -d \
  --name mysql-production \
  -v mysql_data:/var/lib/mysql \
  -v /opt/mysql/conf.d:/etc/mysql/conf.d \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0 \
  --innodb_buffer_pool_size=2G \
  --innodb_log_file_size=512M \
  --max_connections=500 \
  --innodb_flush_log_at_trx_commit=1 \
  --innodb_flush_method=O_DIRECT \
  --innodb_file_per_table=ON \
  --innodb_read_io_threads=8 \
  --innodb_write_io_threads=4 \
  --table_open_cache=2000
```

## 监控与维护

### 监控 MySQL 状态
```bash
# 查看进程列表
docker exec mysql-server mysql -uroot -ppassword -e "SHOW PROCESSLIST"

# 查看状态变量
docker exec mysql-server mysql -uroot -ppassword -e "SHOW GLOBAL STATUS"

# 查看性能指标
docker exec mysql-server mysql -uroot -ppassword -e "SHOW ENGINE INNODB STATUS"
```

### 日志管理
```bash
# 查看错误日志
docker logs mysql-server

# 查看慢查询日志 (需先配置)
docker exec mysql-server cat /var/log/mysql/mysql-slow.log

# 实时监控日志
docker logs -f mysql-server
```

## Docker Compose 配置

### 完整 MySQL 服务配置
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql-server
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: app_db
      MYSQL_USER: app_user
      MYSQL_PASSWORD: user_password
    volumes:
      - mysql_data:/var/lib/mysql
      - ./conf.d:/etc/mysql/conf.d
      - ./logs:/var/log/mysql
    ports:
      - "3306:3306"
    networks:
      - db-network
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    networks:
      - db-network
    depends_on:
      - mysql

networks:
  db-network:
    driver: bridge

volumes:
  mysql_data:
    driver: local
```

## 高可用方案

### 主从复制配置
```yaml
# docker-compose-master-slave.yml
version: '3.8'

services:
  mysql-master:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_REPLICATION_USER: repl_user
      MYSQL_REPLICATION_PASSWORD: repl_pass
    volumes:
      - mysql_master_data:/var/lib/mysql
    ports:
      - "3306:3306"
    command: --server-id=1 --log-bin=mysql-bin --binlog-format=ROW

  mysql-slave:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_MASTER_HOST: mysql-master
      MYSQL_MASTER_USER: repl_user
      MYSQL_MASTER_PASSWORD: repl_pass
      MYSQL_MASTER_PORT: 3306
    volumes:
      - mysql_slave_data:/var/lib/mysql
    depends_on:
      - mysql-master
    command: --server-id=2

volumes:
  mysql_master_data:
  mysql_slave_data:
```

## 故障排查

### 常见问题解决
```bash
# 容器无法启动
docker logs mysql-server

# 连接问题检查
docker exec mysql-server mysqladmin -uroot -ppassword ping

# 修复损坏的表
docker exec mysql-server mysqlcheck -uroot -ppassword --auto-repair --check --all-databases

# 重置 root 密码
docker exec -it mysql-server mysql -uroot -p
# 然后在 MySQL 中执行:
# ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

### 性能问题诊断
```bash
# 查看运行参数
docker exec mysql-server mysql -uroot -ppassword -e "SHOW VARIABLES;"

# 查看运行状态
docker exec mysql-server mysql -uroot -ppassword -e "SHOW STATUS;"

# 查看当前查询
docker exec mysql-server mysql -uroot -ppassword -e "SHOW FULL PROCESSLIST;"
```

> 提示：生产环境部署 MySQL 容器时，务必配置适当的数据持久化、资源限制和定期备份策略。

!> 重要：MySQL root 密码应通过安全方式管理，避免在命令行或脚本中明文存储。