# Linux用户组和权限管理

## 1、linux安全模型

### 1、用户

Linux中每个用户是通过UID(User Id)来标识的

​ 管理员： root， 0

​ 普通用户： 1-60000自动分配

​ 系统用户：1-499 （centos 6以前），1-999（centos 7以后）

​ 对守护进程获取资源进行权限分配

​ 登录用户：500+ （centos6以前），1000+（centos7以后）

​ 给用户进行交互式登录使用

### 2、用户组

Linux中可以将一个或多个用户加入用户组中，用户组是通过GID （Group ID）来标识的

​ 管理员组：root, 0

​ 普通组：

​ 系统组：1-499（centos6以前），1-999（centos7以后），对守护进程获取资源进行权限分配

​ 普通组：500+（centos6以前），1000+（centos7以后），给用户使用

### 3、用户和组的关系

用户的主要组（primary group）：用户必须属于一个主组，默认创建用户时会自动创建和用户名同名的组，做为用户的主要组，由于此组中只有一个用户，又称为私有组

用户的附加组（supplementary group）：一个用户可以属于零个或多个辅助组，附属组

范例：
```bash
[root@jiang ~]# id postfix
uid=89(postfix) gid=89(postfix) groups=89(postfix),12(mail)
```

### 4、安全上下文

Linux安全上下文Context：运行中的程序，

比如：分别以root和jiang的身份运行 /bin/cat /etc/shadow，得到的结果是不同的，资源能否被访问，是由运行者的身份决定，非程序本身

## 2、用户和组的配置文件

### 1、用户和组的主要配置文件

/etc/passwd；用户及其属性信息（名称、UID、主组ID等）

/etc/shadow：用户密码及其相关属性

/etc/group: 组及其属性信息

/etc/gshadow: 组密码及其相关属性

### 2、passwd文件格式

login name：登录用名 （jiang）

passwd：密码（x）

UID：用户身份（1000）

GID：登录默认所在组编号（1000）

GECOS：用户全名或注释

home directory：用户主目录（/home/jiang）

shell：用户默认使用shell（/bin/bash）

### 3、shadow文件格式

登陆用名

用户密码；一般用sha512加密

从1970年1月1日起到密码最近一次被更改的时间

密码再过几天可以被变更（0表示随时可以变更）

密码再过几天必须变更（99999表示永不过期）

密码过期前几天系统提醒用户（默认为一周）

密码过期几天后账号会被锁定

从1970年1月1日算起，多少天后账号失效

更改密码加密算法：

​ authconfig --passalgo=sha256 --update

密码的安全策略

​ 足够长

​ 使用数字、大小写字母及他叔字符中至少3种

​ 使用随机密码

​ 定期更换，不要使用最近使用过的密码

范例：生成随机密码

```bash
[root@jiang ~]# tr -dc '[alum:]' < /dev/urandom | head -c 12
uu[l]m]:u[ll[root@jiang ~]# openssl rand -base64 9
TbOiMDcmKgbB
```

### 4、group文件格式

群组名称：就是群组名称

群组密码：通常不需要设定，密码式被记录在 /etc/gshadow

GID：就是群组的ID

以当前组为附加组的用户列表（分隔符为逗号）

### 5、gshow文件格式

群组名称：就是群的名称

群组密码：

组管理列表：组管理员的列表，更改组密码和成员

以当前组为附加组的用户列表：多个用户间逗号分隔

### 6、文件操作

vipw和vigr

pwc和grpck

## 3、用户和组管理命令

### 3.1、用户创建

​ useradd命令可以创建新的linux用户

格式：

​ useradd [option] LOGIN

常见的选项：

```bash
-u：	UID
-o：	配合-u选项，不检查UID的唯一性
-g：	GID   指明用户所属基本组，可为组名，也可以GID
-c：	"COMMENT"	用户的注释信息
-d：	HOME_DIR    以指定的路径（不存在）为家目录
-s：	SHELL      指明用户的默认shell的程序，可用列表在/etc/shells文件中
-G：	GROUP1[,GROUP2,...]	为用户指明附加组，组须事先存在
-N：	不创建私用组做主组，使用users组做主组
-r:	创建私用组做主组，使用user组做主组
-m:	创建家目录，用于系统用户
-M：	不创建家目录，用于非系统用户
-p：	指定加密的密码
```

范例：

```bash
useradd -r -u 48 -g apache -s /sbin/nologin -d /var/www -c "Apache" apache
```

useradd 命令默认值设定由/etc/default/useradd定义

```bash
[root@jiang ~]# cat /etc/default/useradd
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

​ 显示或更改默认设置

```bash
useradd  -D
useradd  -D -s SHELL
useradd  -D -b BASE_DIR
useradd  -D -g  GROUP
```

新建用户的相关文件

/etc/default/useradd

​ /etc/skel/*

​ /etc/login.defs

批量创建用户

newusers passwd 格式文件

批量修改用户口令

echo username ： passwd | chapassswd

范例:centos 6创建并指定基于sha512的用户密码

范例：利用python程序在Centos 7生成sha512加密密码

范例；Centos 8生成sha512加密密码

```bash
[root@centos8 ~]# openssl passwd -6 magedu
$6$z0Hza3BjGAVbmS04$GaFvomXlcQ/.Td.ZEwArW4evQWsk73mA6MnR3zROmKJNmTgLTNU0giFC2su5w5UHUTl/U9zRw6l.9lLN7mguc0
```

### 3.2、用户属性修改

```bash
usermod命令可以修改用户属性
```

格式：

usermod [OPTION] login

常见选项：

```bash
-u  UID：新UID
-g  GID:	新主组
-G  GROUP1[,GROUP2,...[,GROUPN]]]:	新附加组，原来的附加组将会被覆盖：若保留原有，则需要同时使用-a选项
-s	SHELL：新的默认SHELL
-c	'COMMENT'：新的注释信息
-d	HOME：新家目录不会自动创建，若要创建新家目录并移动原来家数据，同时使用-m选项
-l		login_name：新的名字
-L：lock指定用户，在/etc/shadow  密码栏的增加  ！
-U：	unlock指定用户，将  /etc/shadow 密码栏的 ！拿掉
-e：YYY-MM-DD:指明用户账号过期日期
-f   INACYIVE：设定非活动期限，即宽限期
```

### 3.3、 删除用用户

userdel可删除Linux用户

格式：

```bash
userdel  [OPTION]... Login
```

常见选项：

```bash
-f，--force		强制
-r，--remove	删除用户家目录和邮箱
```

### 3.4、查看当前用户相关的ID信息

id命令可以查看用户的UID，GID等信息

```bash
id  [OPTION]... [USER]
```

常见选项：

```bash
-u:    显示UID
-g：   显示GID
-G：   显示用户所属组的ID
-n：   显示名称，需配合ugG使用
```

### 3.5、切换用户或以其它用户身份执行命令

su:即switch user，命令可以切换用户身份，并以指定用户的身份执行命令

格式:

su [option...] [-] [user [args...]]

常见选项：

-l --login su -l UserName 相当于 su - UserName

-c --command pass a single command to the shell with -c

切换用户的方式：

​ su UserName:非登录式切换，即不会读取用户目标配置文件，不改变当前工作目录，即不完全切换

​ su - UserName：登录式切换，会读取目标用户的配置文件，切换至家目录，即完全切换

说明:root su至其他用户无需密码；非root用户切换时需要密码

注意：su 切换新用户后，使用exit退回至旧的用户，而不要再用su切换旧用户，否则会生成很多的bash子进程，环境可能会混乱。

换个身份执行命令；

```bash
su [-] UserName -c 'COMMAND'
```

范例；

```bash
[root@jiang ~]# getent passwd jiang
jiang:x:1000:1000:jiang:/home/jiang:/bin/bash
[root@jiang ~]# usermod -s /bin/false jiang
[root@jiang ~]# getent passwd jiang
jiang:x:1000:1000:jiang:/home/jiang:/bin/false
[root@jiang ~]# su - jiang
Last login: Tue Dec  8 11:50:29 CST 2020 on pts/1
[root@jiang ~]# whoami
root
```

范例：

```bash
[root@jiang ~]# su -s /sbin/nologin jiang
This account is currently not available.
[root@jiang ~]# whoami
root
[root@jiang ~]# su -s /bin/false jiang
[root@jiang ~]# whoami
root
```

范例：

```bash
[root@jiang ~]# su - root -c "getent shadow"
```

范例；

```bash
[root@jiang ~]# su - jiang -c 'touch jiang.txt'
[root@jiang ~]# ll ~jiang/
total 0
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Desktop
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Documents
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Downloads
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Music
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Pictures
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Public
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Templates
drwxr-xr-x. 2 jiang jiang 6 Dec  1 12:42 Videos
-rw-rw-r--. 1 jiang jiang 0 Dec  8 11:50 jiang.txt
```

范例：

```bash
[root@jiang ~]# su bin
This account is currently not available.
[root@jiang ~]# su -s /bin/bash bin
bash-4.2$ whoami
bin

[root@jiang ~]# getent passwd tss
tss:x:59:59:Account used by the trousers package to sandbox the tcsd 	daemon:/dev/null:/sbin/nologin

root@jiang ~]# su - -s /bin/bash tss
Last login: Tue Dec  8 11:55:02 CST 2020 on pts/1
su: warning: cannot change directory to /dev/null: Not a directory
-bash: /dev/null/.bash_profile: Not a directory
-bash-4.2$ logout
-bash: /dev/null/.bash_logout: Not a directory
```

### 3.6、设置密码

passwd可以修改用户密码

格式：

```bash
passwd [OPTIONS] UserName
```

常用选项：

```bash
-d:删除指定用户密码
-l:锁定指定用户
-u:解锁指定用户
-e:强制用户下次登录
-f:强制操作
-n   mindays:	指定最短使用期限
-x  maxdays：	最大使用期限
-w  warndays：	提前多少天开始警告
-i  inactivedays：非活动期限
--stdin：从标准输入接受用户密码，Ubuntu五次选项
```

### 3.7、修改用户密码策略

chage 可以修改用户密码策略

格式：

```bash
chage [OPTION]...  LOGIN
```

常见选项：

```bash
-d：	LAST_DAY			#更改密码的时间
-m：   --mindays  MIN_DAYS
-M     --maxdays   MAX_DAYS
-w     --warndays   WARN_DAYS
-I     --inactive    INACTIVE		   #密码过期后的宽限期
-E     --expiredate   EXPIRE_DATE           #用户的有效期
-l		显示密码策略
```

范例：

```bash
[root@jiang ~]# chage -m 3 -M 42 -W 14 -I 7 -E 2020-12-8 jiang
[root@jiang ~]# chage -l jiang
Last password change					: never
Password expires					: never
Password inactive					: never
Account expires						: Dec 08, 2020
Minimum number of days between password change		: 3
Maximum number of days between password change		: 42
Number of days of warning before password expires	: 14
[root@jiang ~]# getent shadow jiang
jiang:$6$NBi3RlGoX85VrzH6$Gh.EPK6zRQA01LGHUHvkphaPVquPKqP3u/Qx1m7M64Y9Jw3mWI21vfi6xpe4jMFU0egq6yZvSRQbK7xHBdKi91::3:42:14:7:18604:
[root@jiang ~]# chage -d 0 jiang
[root@jiang ~]# getent shadow jiang
jiang:$6$NBi3RlGoX85VrzH6$Gh.EPK6zRQA01LGHUHvkphaPVquPKqP3u/Qx1m7M64Y9Jw3mWI21vfi6xpe4jMFU0egq6yZvSRQbK7xHBdKi91:0:3:42:14:7:18604:
[root@jiang ~]# chage -d 0 jiang
[root@jiang ~]# getent shadow jiang
jiang:$6$NBi3RlGoX85VrzH6$Gh.EPK6zRQA01LGHUHvkphaPVquPKqP3u/Qx1m7M64Y9Jw3mWI21vfi6xpe4jMFU0egq6yZvSRQbK7xHBdKi91:0:3:42:14:7:18604:
[root@jiang ~]# chage -l jiang
Last password change					: password must be changed
Password expires					: password must be changed
Password inactive					: password must be changed
Account expires						: Dec 08, 2020
Minimum number of days between password change		: 3
Maximum number of days between password change		: 42
Number of days of warning before password expires	: 14
[root@jiang ~]# getent shadow jiang
jiang:$6$NBi3RlGoX85VrzH6$Gh.EPK6zRQA01LGHUHvkphaPVquPKqP3u/Qx1m7M64Y9Jw3mWI21vfi6xpe4jMFU0egq6yZvSRQbK7xHBdKi91:0:3:42:14:7:18604:
```

### 3.8、用户相关的其他命令

cnfn 指定个人信息

chsh 指定shell，相当于usermod -s

finger可看用户个人信息

范例：

```bash
[root@jiang ~]# chfn jiang
Changing finger information for jiang.
Name [jiang]: jiangshiqian
Office []: it
Office Phone []: 10000
Home Phone []: 11111

Finger information changed.
[root@jiang ~]# getent passwd jiang
jiang:x:1000:1000:jiangshiqian,it,10000,11111:/home/jiang:/bin/false

[root@jiang ~]# chsh -s /bin/csh jiang
Changing shell for jiang.
Shell changed.
[root@jiang ~]# getent passwd jiang
jiang:x:1000:1000:jiangshiqian,it,10000,11111:/home/jiang:/bin/csh
[root@jiang ~]# usermod -s /bin/bash jiang
[root@jiang ~]# getent passwd jiang
jiang:x:1000:1000:jiangshiqian,it,10000,11111:/home/jiang:/bin/bash
```

范例：修改用户使用不可登录的shell类型

```bash
[root@jiang ~]# getent passwd jiang
jiang:x:1000:1000:jiangshiqian,it,10000,11111:/home/jiang:/bin/bash
[root@jiang ~]# chsh -s /sbin/nologin jiang
Changing shell for jiang.
chsh: Warning: "/sbin/nologin" is not listed in /etc/shells.
Shell changed.
[root@jiang ~]# su - jiang
Last login: Tue Dec  8 14:01:04 CST 2020 on pts/2
This account is currently not available.
[root@jiang ~]# chsh -s /bin/false jiang
Changing shell for jiang.
chsh: Warning: "/bin/false" is not listed in /etc/shells.
Shell changed.
[root@jiang ~]# su - jiang
Last login: Tue Dec  8 14:47:22 CST 2020 on pts/2
[root@jiang ~]# id
uid=0(root) gid=0(root) groups=0(root) 		context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@jiang ~]# chsh -s /bin/bash jiang
Changing shell for jiang.
Shell changed.
[root@jiang ~]# su - jiang
Last login: Tue Dec  8 14:48:06 CST 2020 on pts/2
```

### 3.9、创建组

groupadd实现创建组

格式

```bash
groupadd [OPTION]... group_name
```

常见选项：

-g GID 指明GID号；[GID_MIN,GID_MAX]

-r 创建系统组，Centos 6之前：ID<500，Centos 7以后：ID<1000

范例：

```bash
[jiang@jiang ~]$ groupadd -g 48 -r apache
```

### 3.10、修改组

groupmod组属性修改

格式:

```bash
groupmod [OPTION]... group
```

常见的选项:

```bash
-n group_name:新名字
-g GID：新的GID
```

### 3.11、组删除

groupdel可以删除组

格式：

```bash
groupdel [options] GROUP
```

常见的选项：

```bash
-f,--force强制删除，即使是用户的主组也强制删除组
```

### 3.12、更改组密码

gpasswd命令，可以更改组密码，也可以修改附加组的成员关系

格式：

```bash
gpasswd [OPTION] GROUP
```

常见的选项：

```bash
-a	user	将user添加至指定组中
-d	user	从指定附加组中移除用户user
-A	user，user2，...设置由管理权限的用户列表
```

范例：

### 3.13 增加组成员 ##################

```bash
[root@jiang ~]# groupadd admins
[root@jiang ~]# id jiang
uid=1000(jiang) gid=1000(jiang) groups=1000(jiang)
[root@jiang ~]# gpasswd -a jiang admins
Adding user jiang to group admins
[root@jiang ~]# id jiang
uid=1000(jiang) gid=1000(jiang) groups=1000(jiang),1001(admins)
[root@jiang ~]# groups jiang
jiang : jiang admins
[root@jiang ~]# getent group admins
admins:x:1001:jiang
```

### 3.14 删除组成员 ######################

```bash
[root@jiang ~]# gpasswd -d jiang admins
Removing user jiang from group admins
[root@jiang ~]# groups jiang
jiang : jiang
[root@jiang ~]# id jiang
uid=1000(jiang) gid=1000(jiang) groups=1000(jiang)
[root@jiang ~]# getent group admins
admins:x:1001:
```

### 3.13、临时切换主组

newgrp命令可以临时切换主组，如果用户本不属于此组，则需要密码

格式：

```bash
newgrp [-] [group]
```

如果使用-选项，可以初始化用户环境

```bash
[root@jiang ~]# gpasswd root
Changing the password for group root
New Password: 
Re-enter new password: 
[root@jiang ~]# getent gshadow root
root:$6$BXQOMme8Hwc$1l1jWxYf5lxnSoqbgdQgnFmqzneXyWs/47WmTq1sHDJoRq9GZ4yV1obQJAYblT1gT3TOshFD45ga7X.EzDBkO0::
[root@jiang ~]# newgrp root
[root@jiang ~]# id
uid=0(root) gid=0(root) groups=0(root) 	context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@jiang ~]# getent passwd jiang
jiang:x:1000:1000:jiangshiqian,it,10000,11111:/home/jiang:/bin/bash
[root@jiang ~]# touch jiang1.txt
[root@jiang ~]# ll
total 12
-rw-r--r--. 1 root root    0 Dec  8 15:13 jiang1.txt
[root@jiang ~]# exit
exit
[root@jiang ~]# id
uid=0(root) gid=0(root) groups=0(root) 	context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@jiang ~]# touch jiang2.txt
[root@jiang ~]# ll
total 12
-rw-r--r--. 1 root root    0 Dec  8 15:13 jiang1.txt
-rw-r--r--. 1 root root    0 Dec  8 15:13 jiang2.txt
```

### 3.14、更改和查看组成员

groupmems可以管理附加组的成员关系

格式：

```bash
groupmems [options] [action]
```

常见选项：

```bash
-g，	--group  groupname	#更改为指定组（只有root）
-a，	--add  username		#指定用户加入组
-d，	--delete  username	#从组中清楚所有成员
-l，	--list			#显示组成员列表 
```

groups可查看用户关系

格式

查看用户所属组列表

```bash
group [OPTION],[USERNAME]...
```

## 4、文件权限管理

### 4.1、文件所有者和组属性操作

1、设置文件的所有者chown

chown命令可以修改文件的属主，也可以修改文件属组

格式：

```bash
chown [OPTION]...  [OWNER][:[GEROUP]  FILE...
chown [OPTION]... --reference-RFILE FILE...
```

用法说明：

OWENR #只修改所有者
OWNR:GROUP #同时修改所有者和属组
：GROUP #只修改属组，冒号也可以使用 . 替换
--reference=RFILE #参考指定的属性，来修改
-R #递归，此选项慎用，非常危险

范例：

```bash
[root@jiang data]# cp /etc/fstab fi.txt
[root@jiang data]# pwd
/data
[root@jiang data]# ll
total 12
-rw-r--r--. 1 root root 595 Dec  8 15:42 fi.txt
-rw-r--r--. 1 root root   6 Dec  5 17:04 linux.txt
-rw-r--r--. 1 root root   4 Dec  5 17:04 win.txt
[root@jiang data]# chown jiang fi.txt
[root@jiang data]# ll
total 12
-rw-r--r--. 1 jiang root 595 Dec  8 15:42 fi.txt
-rw-r--r--. 1 root  root   6 Dec  5 17:04 linux.txt
-rw-r--r--. 1 root  root   4 Dec  5 17:04 win.txt
```

2、设置文件的属组信息chgrd

chgrp命令可以只修改文件的属组

格式

```bash
chgrp   [OPTION]...   GROUP FILE...

chgrp   [OPTION]...   --reference=RFILE  FILE...
```

-R递归

### 4.2、文件权限

1、文件权限说明

文件的权限主要针对三类对象进行定义

```bash
owner		属主，u
group		属组，g
other		其他，o
```

注意：

```bash
用户的最终权限，是从左到右进行顺序匹配，即，所有者，所属组，其他人，一旦匹配权限立即生效，不再向右查看其权限
r和w权限对root  用户无效
只要所有者，所属组或other三者之一有x权限，root就可以执行
```

每个文件针对每类访问者都定义了三种常用权限

每个文件针对每类访问者都定义了三种权限

```bash
r	Readable
w	Writable
x	excutable
```

对文件的权限：

```bash
r	可使用文件查看类工具，比如：cat，可以获取其内容
w	可修改其内容
x	可以把此类文件提请内核启动为一个进程，即可以执行（运行）此文件（此文件的内容必须是可执行）
```

对目录的权限：

```bash
r	可以使用ls 查看此目录中文件列表
w	可在此目录中创建文件，也可以删除目录中的文件，而和此被删除的文件的权限无关
x	可以cd进入此目录。可以使用ls -l查看此类目录中文件元数据（须配合r权限），属于目录的可访问的最小权限
x	只给目录x权限，不给无执行权限的文件x权限
```

2、

格式

```bash
chmod [OPTION]... MODE[，MODE]... FILE...
chmod [OPTION]...  OCTAL -MODE FILE...
#参数RFILE文件的权限，将FILE的修改为同RFILE
chmod [OPTION]... --reference=RFIL FILE
```

说明：

```bash
MODE；who  opt  permission
who：u，g，o，a
opt：+，-，=
permission：r，w，x

修改指定一类用户的所有权限
u=	g=	o=	ug=	a=	u=，g=

修改指定一类用户某个或某个权限
u+	u-	g+	g-	o+	o-	a+	a-	+	-

-R:递归修改权限
```

范例：

```bash
chmod	u+wx，g-r，o=rx  file
chmod	-R	g+rwx	/testdir
chmod	600	file
```

### 4.3、新建文件和目录的默认权限

umask值可以用来保留在创建文件权限

实现方式：

​ 新建文件的默认权限：666-umask，如果所得结果某位存在执行（奇数）权限，则将其权限+1，偶数不变

​ 新建目录的默认权限：777-umask

​ 非特权用户umask默认是022

​ root的umask默认是022

查看umask

```bash
umask
#模式方式显示
umask -S
#输出可被调用
umask -p
```

​ 修改umask

​ umask #

范例：

```bash
umask  002
umask u=rw，g=r，o=
```

持久保存umask

​ 全局设置：/etc/bashrc

​ 用户设置：-/.bashrc

范例：

```bash
[root@jiang data]# umask
0022
[root@jiang data]# umask -S
u=rwx,g=rx,o=rx
[root@jiang data]# umask -p
umask 0022
```

范例：

```bash
[root@jiang data]# umask
0022
[root@jiang data]# ( umask 666; touch /data/fi.txt)
[root@jiang data]# umask
0022
[root@jiang data]# ll /data/fi.txt
-rw-r--r--. 1 jiang root 595 Dec  8 16:55 /data/fi.txt
```

### 4.4、Linux文件系统上的特殊权限

1、特殊权限SUID

前提:进程有属主和属组；文件有属主和属组

- 任何一个可执行程序问件能不能启动为进程，取决发起者对程序文件是否拥有执行权限
- 启动为进程之后，其进程的属主为发起者，进程的属组为发起者所属的组
- 进程访问文件时的权限，却决于进程的发起者

​ （a）进程的发起者，同文件的属主：则应用文件属主权限

​ （b）进程的发起者，属于文件属组；则应用文件属组权限

​ （c）应用文件“其他”权限

二进制的可执行文件上SUID权限功能：

​ 任何一个可执行程序文件能不能启动为进程：取决发起者对程序文件是否拥有执行权限

​ 启动为进程之后，其进程的属主为原程序文件的属主

​ SUID只对二进制可执行程序有效

​ SUID设置在目录上无意义

SUID权限设定：

```bash
chmod  u+s  FILE...
chmod  4xxx  FILE
chmod  u-s   FILE...
```

范例：

```bash
[root@jiang data]# ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 27856 Apr  1  2020 /usr/bin/passwd
```

2、特殊权限SGID
​ 二进制的可执行文件上SGID权限功能：

​ 任何一个可执行程序文件能不能启动为进程：取决发起者对程序文件是否拥有执行权限

​ 启动为进程之后，其进程的属组为原有程序文件的属组

SGID权限设定：

```bash
chmod   g+s  FILE...
chmod   2xxx  FILE
chmod   g-s  FILE...
```

目录上的SGID权限功能：

默认情况下，用户创建文件时，其属组为此用户所属的主组，一旦某目录被设定了SGID，则对此目录有写权限的用户在次目录中创建的文件所属组为此目录的属组，通常用于创建一个协作目录

SGID权限设定：

```bash
chmod  g+s  DIR...
chmod  2xxx   DIR
chmod   g-s   DIR...
```

3、特殊权限 Stickv 位

具有写权限的目录通常用户可以删除改目录中的任何文件，无论该文件的权限或拥有权

在目录设置Sticky位，只有文件的所有者或root可以删除该文件

sticky权限设定：

```bash
chmod o+t   DIR...
chmod   1xxx   DIR
chmod  o-t	DIR...
```

范例：

```bash
[root@jiang data]# ll -d /tmp
drwxrwxrwt. 26 root root 4096 Dec  8 15:07 /tmp
```

4、特殊权限数字法

SUID SGID STICKY

```bash
000	0
001	1
010	2
011	3
100	4
101	5
110	6
111	7
```

范例：

```bash
chmod  4777  /tmp/a.txt
```

权限位映射

SGID：user，占据属主的执行权限位

​ s: 属主拥有x权限

​ S：属主没有x权限

SGID：group，占据属组的执行权限位

​ s：group拥有x权限

​ S: group没有x权限

Sticky：other，占据other的执行权限位

​ t: other拥有x权限

​ T：other没有x权限

### 4.5、设定文件特殊属性

​ 设置文件的特殊属性，可以访问root用户误操作删除或修改文件

​ 不能删除，改名，更名

​ chattr +i

只能追加内特，不能删除，改名

​ chattr +a

显示特定属性

​ lsattr

​ 4.6、访问控制列表ACL

​ 1、ACL权限功能

ACL：Access Control List，实现灵活的权限管理

除了文件的所有者，所属组和其他人，可以对更多的用户设置权限

Centos 7默认创建的xfs和ext4文件系统具有ACL功能

Centos 7之前版本，默认手工创建的ext4文件系统无ACL功能，需手动增加

```bash
tune2fs  -o  acl  /dev/sdb1
mount  -o  acl  /dev/sdb1    /mnt/test
```

ACL生效顺序：

```bash
所有者，自定义用户，所属组 | 自定义组，其他人
```

​ 2、ACL相关命令

setfacl 可设置ACL权限

getfacl 可查看设置的ACL权限

范例：

```bash
[root@jiang data]# ll f1.txt
-rw-r--r--. 1 root root 0 Dec  8 19:27 f1.txt
[root@jiang data]# setfacl -m u:jiang:- f1.txt
[root@jiang data]# ll
total 12
-rw-r--r--. 1 root  root   0 Dec  8 17:46 dir
-rw-r--r--+ 1 root  root   0 Dec  8 19:27 f1.txt
-rw-r--r--. 1 jiang root 595 Dec  8 16:55 fi.txt
-rw-r--r--. 1 root  root   6 Dec  5 17:04 linux.txt
-rw-r--r--. 1 root  root   4 Dec  5 17:04 win.txt
[root@jiang data]# su jiang
[jiang@jiang data]$ cat f1.txt
cat: f1.txt: Permission denied
[jiang@jiang data]$ echo xx >> f1.txt
bash: f1.txt: Permission denied
```

范例：

```bash
[root@jiang data]# ll f1.txt
-rw-r--r--+ 1 root root 0 Dec  8 19:27 f1.txt
[root@jiang data]# setfacl -m u:jiang:- f1.txt
[root@jiang data]# ll
total 12
-rw-r--r--. 1 root  root   0 Dec  8 17:46 dir
-rw-r--r--+ 1 root  root   0 Dec  8 19:27 f1.txt
-rw-r--r--. 1 jiang root 595 Dec  8 16:55 fi.txt
-rw-r--r--. 1 root  root   6 Dec  5 17:04 linux.txt
-rw-r--r--. 1 root  root   4 Dec  5 17:04 win.txt
[root@jiang data]# getfacl f1.txt
# file: f1.txt
# owner: root
# group: root
user::rw-
user:jiang:---
group::r--
mask::r--
other::r--

[root@jiang data]# su jiang
[jiang@jiang data]$ cat f1.txt
cat: f1.txt: Permission denied
[jiang@jiang data]$ echo xx >> f1.txt
bash: f1.txt: Permission denied
```

mask权限

​ mask只影响除所有者和other的之外的人和组的最大权限

​ mask需要与用户的权限进行逻辑与运算后，才能变成有限的权限（Effective Permission）

​ 用户或组的设置必须存在于mask权限设定范围内才会生效

范例：

setfacl -m mask：：rx file

范例：

```bash
[root@jiang data]# ll f1.txt
-rw-rw-r--+ 1 root root 0 Dec  8 19:27 f1.txt
[root@jiang data]# chmod g-r f1.txt
[root@jiang data]# ll f1.txt
-rw--w-r--+ 1 root root 0 Dec  8 19:27 f1.txt
[root@jiang data]# getfacl f1.txt
# file: f1.txt
# owner: root
# group: root
user::rw-
user:jiang:---
group::r--			#effective:---
group:admins:-w-
mask::-w-
other::r--

[root@jiang data]# setfacl -m mask::rw f1.txt
[root@jiang data]# getfacl f1.txt
# file: f1.txt
# owner: root
# group: root
user::rw-
user:jiang:---
group::r--
group:admins:-w-
mask::rw-
other::r--

[root@jiang data]# setfacl -m u:jiang:rwx f1.txt
[root@jiang data]# getfacl f1.txt
# file: f1.txt
# owner: root
# group: root
user::rw-
user:jiang:rwx
group::r--
group:admins:-w-
mask::rwx
other::r--

[root@jiang data]# setfacl -m mask::rw f1.txt
[root@jiang data]# getfacl f1.txt
# file: f1.txt
# owner: root
# group: root
user::rw-
user:jiang:rwx			#effective:rw-
group::r--
group:admins:-w-
mask::rw-
other::r--
```

set选项会把原有的ACl都删除，用新的替代，需要注意的时一定要包含UGO的设置，不能像-m一样只是添加ACl就可以

范例：

setfacl --set u::rw,u:jiang:rw,g::r,o::- faile1

​ 3、备份和还原ACL

主要文件操作命令cp和mv都支持ACl，只是cp命令需要加上-p参数，但是tar等常见的备份工具是不会保留目录和文件的ACl信息

范例：

```bash
#备份ACL
getfac1 -R /tmp/dir > ac1. txt
#消除ACL权限
setfac1 -R -b /tmp/dir
#还，原ACL权限
setfacl -R --set-file=acl.txt / tmp/dir
#还原ACL权限
setfac1 --restore ac1. txt
#查看ACL权限
getfac1 -R / tmp/dir
```

范例：

```bash
[root@jiang data]# getfacl *
# file: dir
# owner: root
# group: root
user::rw-
group::r--
other::r--

# file: f1.txt
# owner: root
# group: root
user::rw-
user:jiang:rwx			#effective:rw-
group::r--
group:admins:-w-
mask::rw-
other::r--

# file: fi.txt
# owner: jiang
# group: root
user::rw-
group::r--
other::r--

# file: linux.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--

# file: win.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--

[root@jiang data]# cd
[root@jiang ~]# tar cvf data.tar /data/
tar: Removing leading `/' from member names
/data/
/data/linux.txt
/data/win.txt
/data/fi.txt
/data/dir
/data/f1.txt
[root@jiang ~]# tar xvf data.tar -C /opt
data/
data/linux.txt
data/win.txt
data/fi.txt
data/dir
data/f1.txt
[root@jiang ~]# ls /opt
data  rh
[root@jiang ~]# cd /opt/data
[root@jiang data]# ll
total 12
-rw-r--r--. 1 root  root   0 Dec  8 17:46 dir
-rw-rw-r--. 1 root  root   0 Dec  8 19:27 f1.txt
-rw-r--r--. 1 jiang root 595 Dec  8 16:55 fi.txt
-rw-r--r--. 1 root  root   6 Dec  5 17:04 linux.txt
-rw-r--r--. 1 root  root   4 Dec  5 17:04 win.txt
[root@jiang data]# getfacl -R /data > /root/acl.txt
getfacl: Removing leading '/' from absolute path names
[root@jiang data]# cat /root/acl.txt
# file: data
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

# file: data/linux.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--

# file: data/win.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--

# file: data/fi.txt
# owner: jiang
# group: root
user::rw-
group::r--
other::r--

# file: data/dir
# owner: root
# group: root
user::rw-
group::r--
other::r--

# file: data/f1.txt
# owner: root
# group: root
user::rw-
user:jiang:rwx	#effective:rw-
group::r--
group:admins:-w-
mask::rw-
other::r--

[root@jiang data]# ll /opt/data
total 12
-rw-r--r--. 1 root  root   0 Dec  8 17:46 dir
-rw-rw-r--. 1 root  root   0 Dec  8 19:27 f1.txt
-rw-r--r--. 1 jiang root 595 Dec  8 16:55 fi.txt
-rw-r--r--. 1 root  root   6 Dec  5 17:04 linux.txt
-rw-r--r--. 1 root  root   4 Dec  5 17:04 win.txt
[root@jiang data]# cd
[root@jiang ~]# setfacl -R --set-file=/root/acl.txt /opt
[root@jiang ~]# ll /opt/data/
total 12
-rw-rw-r--+ 1 root  root   0 Dec  8 17:46 dir
-rw-rw-r--+ 1 root  root   0 Dec  8 19:27 f1.txt
-rw-rw-r--+ 1 jiang root 595 Dec  8 16:55 fi.txt
-rw-rw-r--+ 1 root  root   6 Dec  5 17:04 linux.txt
-rw-rw-r--+ 1 root  root   4 Dec  5 17:04 win.txt
[root@jiang ~]# setfacl -b -R /opt/data
[root@jiang ~]# ll /opt/data
total 12
-rw-r--r--. 1 root  root   0 Dec  8 17:46 dir
-rw-r--r--. 1 root  root   0 Dec  8 19:27 f1.txt
-rw-r--r--. 1 jiang root 595 Dec  8 16:55 fi.txt
-rw-r--r--. 1 root  root   6 Dec  5 17:04 linux.txt
-rw-r--r--. 1 root  root   4 Dec  5 17:04 win.txt
```
