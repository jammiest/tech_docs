# Linux 命令速查表

## 文件与目录操作

### 基本导航
```bash
pwd                         # 显示当前工作目录
ls                          # 列出目录内容
ls -l                       # 详细列表格式
ls -a                       # 显示所有文件（包括隐藏文件）
ls -lh                      # 人类可读的文件大小
cd /path/to/directory       # 切换目录
cd ~                        # 切换到用户主目录
cd -                        # 切换到上一个目录
```

### 文件操作
```bash
touch filename              # 创建空文件
mkdir directory             # 创建目录
mkdir -p dir1/dir2/dir3     # 创建多级目录
cp file1 file2              # 复制文件
cp -r dir1 dir2             # 递归复制目录
mv file1 file2              # 移动/重命名文件
rm file                     # 删除文件
rm -r directory             # 递归删除目录
rm -rf directory            # 强制递归删除（谨慎使用）
```

### 文件查看
```bash
cat file                    # 显示文件全部内容
less file                   # 分页查看文件（可向前向后）
more file                   # 分页查看文件（只能向前）
head -n 10 file             # 显示文件前10行
tail -n 10 file             # 显示文件后10行
tail -f file                # 实时跟踪文件变化
```

## 文件权限与属性

### 权限管理
```bash
chmod 755 file              # 更改文件权限（rwxr-xr-x）
chmod u+x file              # 给所有者添加执行权限
chmod g-w file              # 移除组用户的写权限
chmod o=r file              # 设置其他用户只读权限
chown user:group file       # 更改文件所有者和组
chgrp group file            # 更改文件所属组
```

### 文件信息
```bash
stat file                   # 显示文件详细信息
file filename               # 显示文件类型
du -sh directory            # 显示目录大小
df -h                       # 显示磁盘空间使用情况
```

## 文本处理

### 搜索与过滤
```bash
grep "pattern" file         # 在文件中搜索文本
grep -r "pattern" directory # 递归搜索目录
grep -i "pattern" file      # 忽略大小写搜索
grep -v "pattern" file      # 反向搜索（不匹配的行）
find /path -name "*.txt"    # 按名称查找文件
find /path -type f -mtime -7 # 查找7天内修改的文件
```

### 文本处理工具
```bash
awk '{print $1}' file       # 打印每行的第一个字段
sed 's/old/new/g' file      # 替换文本
sort file                   # 排序文件内容
sort -r file                # 反向排序
uniq file                   # 去除重复行
wc -l file                  # 统计行数
wc -w file                  # 统计单词数
wc -c file                  # 统计字节数
```

## 系统信息

### 系统状态
```bash
uname -a                    # 显示系统信息
hostname                    # 显示主机名
whoami                      # 显示当前用户名
who                         # 显示登录用户
w                           # 显示系统负载和登录用户
uptime                      # 显示系统运行时间
```

### 进程管理
```bash
ps aux                      # 显示所有进程
ps -ef                      # 显示完整格式的进程信息
top                         # 实时显示进程状态
htop                        # 增强版的top（需要安装）
kill PID                    # 终止进程
kill -9 PID                 # 强制终止进程
pkill process_name          # 按名称终止进程
```

### 硬件信息
```bash
free -h                     # 显示内存使用情况
df -h                       # 显示磁盘空间
du -sh *                    # 显示当前目录各文件大小
lscpu                       # 显示CPU信息
lsblk                       # 显示块设备信息
```

## 网络相关

### 网络配置
```bash
ifconfig                    # 显示网络接口信息
ip addr show                # 显示IP地址（现代替代ifconfig）
ping example.com            # 测试网络连接
traceroute example.com      # 跟踪网络路径
netstat -tulpn              # 显示网络连接和端口
ss -tulpn                   # 更快的netstat替代
```

### 网络工具
```bash
curl http://example.com     # 传输URL数据
wget http://example.com/file # 下载文件
ssh user@host               # 安全shell连接
scp file user@host:/path    # 安全复制文件
rsync -av source/ dest/     # 同步文件和目录
```

## 压缩与归档

### 压缩解压
```bash
tar -czvf archive.tar.gz directory  # 创建gzip压缩包
tar -xzvf archive.tar.gz           # 解压gzip压缩包
tar -cjvf archive.tar.bz2 directory # 创建bzip2压缩包
tar -xjvf archive.tar.bz2          # 解压bzip2压缩包
zip -r archive.zip directory       # 创建zip压缩包
unzip archive.zip                  # 解压zip压缩包
gzip file                          # 压缩文件
gunzip file.gz                     # 解压文件
```

## 包管理

### Debian/Ubuntu (APT)
```bash
apt update                   # 更新包列表
apt upgrade                  # 升级所有包
apt install package          # 安装包
apt remove package           # 删除包
apt autoremove               # 删除不需要的包
apt search pattern           # 搜索包
apt show package             # 显示包信息
```

### RedHat/CentOS (YUM/DNF)
```bash
yum update                   # 更新所有包
yum install package          # 安装包
yum remove package           # 删除包
yum search pattern           # 搜索包
dnf update                   # DNF（YUM的下一代）
```

## 用户管理

### 用户操作
```bash
whoami                       # 显示当前用户
id                           # 显示用户ID信息
sudo command                 # 以超级用户权限执行命令
su - username                # 切换用户
useradd username             # 添加用户
usermod -aG group username   # 将用户添加到组
userdel username             # 删除用户
passwd username              # 更改用户密码
```

### 组管理
```bash
groups username              # 显示用户所属组
groupadd groupname           # 添加组
groupdel groupname           # 删除组
```

## 系统管理

### 服务管理
```bash
systemctl start service      # 启动服务
systemctl stop service       # 停止服务
systemctl restart service    # 重启服务
systemctl status service     # 查看服务状态
systemctl enable service     # 设置开机启动
systemctl disable service    # 禁用开机启动
journalctl -u service        # 查看服务日志
```

### 任务调度
```bash
crontab -e                   # 编辑cron任务
crontab -l                   # 列出cron任务
crontab -r                   # 删除所有cron任务
at now + 1 hour              # 一次性定时任务
```

## 性能监控

### 系统监控
```bash
top                          # 实时系统监控
htop                         # 增强版top
iotop                        # I/O监控
iftop                        # 网络流量监控
vmstat 1                     # 虚拟内存统计
iostat 1                     # I/O统计
sar                          # 系统活动报告
```

### 日志查看
```bash
dmesg                        # 查看内核日志
dmesg -T                     # 带时间戳的内核日志
journalctl -f                # 实时查看系统日志
tail -f /var/log/syslog      # 实时查看系统日志
tail -f /var/log/auth.log    # 实时查看认证日志
```

## 快捷键与技巧

### 终端快捷键
```bash
Ctrl + C                     # 终止当前命令
Ctrl + Z                     # 暂停当前命令
Ctrl + D                     # 退出终端或发送EOF
Ctrl + L                     # 清屏
Ctrl + A                     # 移动到行首
Ctrl + E                     # 移动到行尾
Ctrl + U                     # 删除到行首
Ctrl + K                     # 删除到行尾
Ctrl + R                     # 搜索命令历史
Tab                          # 自动补全
```

### 实用技巧
```bash
command > file               # 重定向输出到文件
command >> file              # 追加输出到文件
command 2> file              # 重定向错误输出
command &> file              # 重定向所有输出
command1 | command2          # 管道，将第一个命令的输出作为第二个命令的输入
nohup command &              # 在后台运行命令，退出终端不终止
(command)                    # 在子shell中执行命令
```

## 环境变量
```bash
echo $PATH                   # 显示PATH环境变量
export VAR=value             # 设置环境变量
unset VAR                    # 删除环境变量
env                          # 显示所有环境变量
set                          # 显示所有变量和函数
```

## 历史命令
```bash
history                      # 显示命令历史
!n                           # 执行历史中第n条命令
!!                           # 执行上一条命令
!string                      # 执行最近以string开头的命令
Ctrl + R                     # 搜索命令历史
```

## 文件比较
```bash
diff file1 file2             # 比较两个文件
diff -u file1 file2          # 统一格式比较
cmp file1 file2              # 逐字节比较文件
md5sum file                  # 计算MD5校验和
sha256sum file               # 计算SHA256校验和
```

这个速查表涵盖了Linux系统管理中最常用的命令，适合日常使用和快速参考。