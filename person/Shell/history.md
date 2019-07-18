# History




## 快捷命令

```shell
# 执行history命令中行号为1997的命令
!1997

# 打印history行号1997的命令，只打印不执行
!1997:p

# 执行最后一条命令行
!!

# 根据给出的命令行部分开头片段 执行该命令
!command_starting_string

# 移除原行号1996的命令，然后下条命令上移到1996行，其他剩余命令依此上移
history -d 1996

# 清除整个history记录
history -c
```


## 管道

```shell
# 查看一页命令历史记录
history | less

# 浏览最后10条命令历史记录
history | tail

# 浏览最后25条命令历史记录
history 25

history | grep chpasswd

history | grep -i searchterm | less
```