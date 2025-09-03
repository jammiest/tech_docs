# 虚拟机 vboxsf 文件夹权限导致 docker 容器中的 nginx 权限不足问题

通过 vboxsf 文件共享系统将本机项目文件共享至虚拟机，虚拟机中通过 docker 容器运行了 nginx 项目，访问项目时返回 404，查看项目 nginx 错误日志：permission denied。

解决：先查询虚拟机 vboxsf 用户组ID：

cat /etc/group

```bash
vboxsf:x:993:nginx,www-data  # 记住 ID 993
```

在 nginx 和 php 的 Dockerfile 文件中添加

> php 的 Dockerfile

```Dockerfile
RUN groupadd --gid 993 vboxsf         # 添加与容器外对应 ID 的用户组
RUN usermod -aG vboxsf www-data   # 将容器运行的用户分配到该组
```

> nginx 的 Dockerfile

```Dockerfile
RUN groupadd --gid 993 vboxsf
RUN usermod -aG vboxsf nginx
```
