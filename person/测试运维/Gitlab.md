# Gitlab实战

> 本文基于 Docker Desktop (WSL) + 极狐18.0进行实战

## gitlab




## gitlab-runner

创建gitlab-runner容器：

```shell
docker run -it -d --name gitlab-runner -v gitlab-runner-config:/etc/gitlab-runner --network docker_default gitlab/gitlab-runner:latest
```

> 如果需要执行删除卷，请执行`docker volume rm gitlab-runner-config`
> `docker volume create gitlab-runner-config`

进入极狐管理后台，新建runner(参考地址：http://gitlab.example.com:8929/admin/runners，点击右上角Create instance runner，随意填写或根据自己情况填写都行，然后创建)

【Register runner -> Step 1】，拿到类似命令
```gitlab-runner register  --url http://gitlab.example.com:8929  --token glrt-NYRx7VQKzAERhpB39QcLQ286MQp0OjEKdToxCw.01.120uc7vav```

进入容器注册runner:

```shell
docker exec -it gitlab-runner bash

# 生成密钥
ssh-keygen -t rsa

gitlab-runner register
# 把拿到的命令填写此处，选择ssh方式

```

### ssh

```shell
ssh-keygen -t rsa -b 4096 -C "testing@gitlab.example.com"
```


/root/.ssh/id_rsa

### PS 重启runner

查看进程PID,使用kill重启进程


`ps aux | grep <进程名>`

# 重启进程

您可以使用系统信号与极狐GitLab Runner 进行交互。以下命令支持以下信号：

| Command         | Signal          | Action                                                             |
| --------------- | --------------- | ------------------------------------------------------------------ |
| `register`        | **SIGINT**          | 取消 runner 注册并删除已注册的 runner。                            |
| `run`, `run-single` | **SIGINT**, **SIGTERM** | 中止所有正在运行的构建并尽快退出。使用两次以立即退出（**强制关闭**）。 |
| `run`, `run-single` | **SIGQUIT**         | 停止接受新构建。当正在运行的构建完成时退出（**正常关闭**）。           |
| `run`             | **SIGHUP**          | 强制重新加载配置文件。                                             |

要强制重新加载 runner 的配置文件，请运行

`kill -SIGHUP <main_runner_pid>`

对于正常关闭：

sudo kill -SIGQUIT <main_runner_pid>

> 不要使用 `killall` 或 `pkill` 进行正常关闭，如果您使用 shell 或 docker 执行器。这可能会导致由于子进程也被杀死而导致信号处理不当。仅在处理作业的主进程上使用它。