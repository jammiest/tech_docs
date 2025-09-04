# KiLL

`killall`是一个基于**名称**终止系统上运行进程的工具。  
`kill`则是终止基于**进程ID号**（PID）的进程。  
kill和killall还可以向进程发送特定的系统信号。

## Killall格式

```bash
killall [process name]
```

### killall用法

killall将终止与指定名称匹配的所有程序。killall发送SIGTERM信号，它终止与指定名称匹配的正在运行的进程。您可以使用以下-s选项指定不同的信号：

```bash
killall -s 9 [process name]
```

这发送SIGKILL信号，您还可以使用以下格式之一指定信号：

```bash
killall -KILL [process name]
killall -SIGKILL [process name]
killall -9 [process name]
```

## Kill格式

kill命令终止其PID指定的各个进程。

```bash
kill [PID]
```

### Kill用法

如果没其他选项，则`kill`发送**SIGTERM**到指定的PID并要求应用程序或服务自行关闭。

在一个`kill`命令中可以指定多个pid和备用系统信号。下面的示例都将`SIGKILL`信号发送到指定的PID：

```bash
kill -s KILL [PID]
kill -KILL [PID]
```

## 系统信号

kill命令不会直接终止进程。相反，一个信号被发送到进程，如果进程接收到一个给定的信号，进程将有相应的指令。手册页提供了所有可用信号的进一步参考:：

```bash
man 7 signal
```

    Standard signals
    Linux supports the standard signals listed below. Several signal numbers are architecture-dependent, as indicated in the "Value" column. Where
    three values are given, the first one is usually valid for alpha and sparc, the middle one for x86, arm, and most other architectures, and the last
    one for mips. (Values for parisc are not shown; see the Linux kernel source for signal numbering on that architecture.) A dash (-) denotes that a
    signal is absent on the corresponding architecture.
    ​
    First the signals described in the original POSIX.1-1990 standard.
    ​
    Signal Value Action Comment
    ──────────────────────────────────────────────────────────────────────
    SIGHUP 1 Term Hangup detected on controlling terminal
    or death of controlling process
    SIGINT 2 Term Interrupt from keyboard
    SIGQUIT 3 Core Quit from keyboard
    SIGILL 4 Core Illegal Instruction
    SIGABRT 6 Core Abort signal from abort(3)
    SIGFPE 8 Core Floating-point exception
    SIGKILL 9 Term Kill signal
    SIGSEGV 11 Core Invalid memory reference
    SIGPIPE 13 Term Broken pipe: write to pipe with no
    readers; see pipe(7)
    SIGALRM 14 Term Timer signal from alarm(2)
    SIGTERM 15 Term Termination signal
    SIGUSR1 30,10,16 Term User-defined signal 1
    SIGUSR2 31,12,17 Term User-defined signal 2
    SIGCHLD 20,17,18 Ign Child stopped or terminated
    SIGCONT 19,18,25 Cont Continue if stopped
    SIGSTOP 17,19,23 Stop Stop process
    SIGTSTP 18,20,24 Stop Stop typed at terminal
    SIGTTIN 21,21,26 Stop Terminal input for background process
    SIGTTOU 22,22,27 Stop Terminal output for background process
    ​
    The signals SIGKILL and SIGSTOP cannot be caught, blocked, or ignored.

简单地列出所有可用的信号，而不包含它们的描述：

```bash
kill -l
killall -l
```

如果需要将信号名称转换为信号编号，或将信号编号转换为信号名称，请使用以下示例：

```bash
$ kill -l 9
KILL
​
$ kill -l kill
9
```

## 查找正在运行的进程

使用像`htop`或`top`这样的实用程序来查看进程的实时列表及其对系统资源的消耗。

使用`ps`命令查看当前正在运行的进程及其`pid`。下面的示例使用`grep`过滤当前为字符串*emacs*运行的所有进程列表:

```bash
$ ps aux | grep "emacs"
username  3896  0.0  2.2  56600 44468 ?        Ss   Sep30   4:29 emacs
username 22843  0.0  0.0   3900   840 pts/11   S+   08:49   0:00 grep emacs
```

左边第二列中列出的数字是PID，在emacs过程中是3896。grep进程总是与自己匹配以进行简单的搜索，就像第二个结果一样。

> 注意: 您可以使用命令`ps auxf`查看所有正在运行的进程的分层树。

获得PID或进程名称后，使用killall或kill终止上述过程。

找到PID的另一个选择是pgrep。

```bash
pgrep [process name]
```

## 验证流程终止

将`-w`选项添加到`killall`命令，将使`killall`等待进程终止后退出。考虑下面的命令：

```bash
killall -w irssi
```

这个示例将SIGTERM系统信号发送给一个后台进程，该进程的名称与irssi匹配。killall将等待匹配的进程结束。如果没有进程匹配指定的名称，killall将返回一条错误消息：

```bash
$ killall -w irssi
irssi: no process found
```
