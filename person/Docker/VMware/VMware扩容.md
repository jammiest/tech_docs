# VMware扩容

## 官方文档

扩展虚拟硬盘：<https://docs.vmware.com/cn/VMware-Workstation-Pro/15.0/com.vmware.ws.using.doc/GUID-29FEE90F-FD7B-44BD-8B52-10C2474CE0AE.html>

## 步骤

### WMware虚拟机调整

扩展虚拟硬盘可增加虚拟机的存储空间。

扩展虚拟硬盘时，新增的空间不会立即提供给虚拟机使用。要让新增空间变为可用，必须使用磁盘管理工具增加虚拟硬盘现有分区的大小，使其与扩展后的大小相匹配。

您所用的磁盘管理工具取决于虚拟机的客户机操作系统。很多操作系统（包括 Windows 7 及更高版本，以及许多版本的 Linux）都提供了可用于调整分区大小的内置磁盘管理工具。另外还有一些第三方磁盘管理工具可供使用，如 Symantec/Norton PartitionMagic、EASEUS Partition Master、Acronis Disk Director 以及开源工具 GParted。

扩展虚拟硬盘大小时，分区和文件系统大小不受影响。

要为选定的虚拟机扩展虚拟硬盘，请选择虚拟机 > 设置，单击硬件选项卡，选择虚拟硬盘，然后从实用工具菜单中选择扩展。

?> 除此之外还有一种扩展方式，即为虚拟机添加新的虚拟硬盘。

### LVM（centOS 7 minimal）调整

注意：扩展磁盘需要在此虚拟机停止的状态下进行，同时扩展的数字是扩展后的预期大小，比如事前为15G，希望扩展20G，应该输入20。这篇文章使用扩展磁盘的方式。

扩展后，重新启动linux，发现df状态没有变化
```bash
[root@service ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   13G   13G   28M  100% /
devtmpfs                 475M     0  475M    0% /dev
tmpfs                    487M     0  487M    0% /dev/shm
tmpfs                    487M  7.9M  479M    2% /run
tmpfs                    487M     0  487M    0% /sys/fs/cgroup
/dev/sda1               1014M  133M  882M   14% /boot
tmpfs                     98M     0   98M    0% /run/user/0
[root@mail ~]#
```

使用fdisk确认磁盘空间是否已经扩展

```bash
[root@service ~]# fdisk -l

磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000cd333

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    31457279    14679040   8e  Linux LVM

磁盘 /dev/mapper/centos-root：13.4 GB, 13417578496 字节，26206208 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：1610 MB, 1610612736 字节，3145728 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

[root@service ~]# 
```

可以看到“Disk /dev/sda: 21.5 GB”，已经扩展了5G空间。

**扩展分区**

```bash
[root@service ~]# fdisk /dev/sda 
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：m
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition\'s system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

命令(输入 m 获取帮助)：n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
分区号 (3,4，默认 3)：
起始 扇区 (31457280-41943039，默认为 31457280)：
将使用默认值 31457280
Last 扇区, +扇区 or +size{K,M,G} (31457280-41943039，默认为 41943039)：
将使用默认值 41943039
分区 3 已设置为 Linux 类型，大小设为 5 GiB

命令(输入 m 获取帮助)：t
分区号 (1-3，默认 3)：
Hex 代码(输入 L 列出所有代码)：L

 0  空              24  NEC DOS         81  Minix / 旧 Linu bf  Solaris        
 1  FAT12           27  隐藏的 NTFS Win 82  Linux 交换 / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 隐藏的 C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux 扩展      c7  Syrinx         
 5  扩展            41  PPC PReP Boot   86  NTFS 卷集       da  非文件系统数据 
 6  FAT16           42  SFS             87  NTFS 卷集       db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux 纯文本    de  Dell 工具      
 8  AIX             4e  QNX4.x 第2部分  8e  Linux LVM       df  BootIt         
 9  AIX 可启动      4f  QNX4.x 第3部分  93  Amoeba          e1  DOS 访问       
 a  OS/2 启动管理器 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad 休 eb  BeOS fs        
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT            
 f  W95 扩展 (LBA)  54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC  
11  隐藏的 FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor      
12  Compaq 诊断     5c  Priam Edisk     a9  NetBSD          f4  SpeedStor      
14  隐藏的 FAT16 <3 61  SpeedStor       ab  Darwin 启动     f2  DOS 次要       
16  隐藏的 FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS    
17  隐藏的 HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE 
18  AST 智能睡眠    65  Novell Netware  b8  BSDI swap       fd  Linux raid 自动
1b  隐藏的 W95 FAT3 70  DiskSecure 多启 bb  Boot Wizard 隐  fe  LANstep        
1c  隐藏的 W95 FAT3 75  PC/IX           be  Solaris 启动    ff  BBT            
1e  隐藏的 W95 FAT1 80  旧 Minix       
Hex 代码(输入 L 列出所有代码)：8e
已将分区“Linux”的类型更改为“Linux LVM”

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘。

[root@service ~]# 
```

**执行 partprobe或者重启**

执行 partprobe命令用于将磁盘分区表变化信息通知内核，并请求操作系统重新加载分区表，可以避免必须重新启动的问题，这里我们reboot一下。

```bash
[root@service ~]# reboot
```

**分区确认**

通过fdisk可以确认到已经添加了sda3

```bash
[root@service ~]# fdisk -l

磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000cd333

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    31457279    14679040   8e  Linux LVM
/dev/sda3        31457280    41943039     5242880   8e  Linux LVM

磁盘 /dev/mapper/centos-root：13.4 GB, 13417578496 字节，26206208 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：1610 MB, 1610612736 字节，3145728 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

[root@service ~]# 
```


**扩展vg**

基本LVM知识，进行vg扩展，不再赘述。

```bash
[root@service ~]# pvcreate /dev/sda3
  Physical volume "/dev/sda3" successfully created.
[root@service ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree
  centos   1   2   0 wz--n- <14.00g    0 
[root@service ~]# vgextend centos /dev/sda3
  Volume group "centos" successfully extended
[root@service ~]# vgs
  VG     #PV #LV #SN Attr   VSize  VFree 
  centos   2   2   0 wz--n- 18.99g <5.00g
[root@service ~]# 
```

**扩展lv**

可以将此lv全部添加或者部分添加，我们这里全部添加。

```bash
[root@service ~]# lvs
  LV   VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root centos -wi-ao---- <12.50g                                                    
  swap centos -wi-ao----   1.50g                                                    
[root@service ~]# lvextend /dev/centos/root /dev/sda3
  Size of logical volume centos/root changed from <12.50 GiB (3199 extents) to 17.49GiB (4478 extents).
  Logical volume centos/root successfully resized.
[root@service ~]# lvs
  LV   VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root centos -wi-ao---- 17.49g                                                    
  swap centos -wi-ao----  1.50g                                                    
[root@service ~]# df

```

**df状态确认**

此时df状态还没有变化

```bash
[root@service ~]# df
文件系统                   1K-块     已用   可用 已用% 挂载点
/dev/mapper/centos-root 13092864 13064864  28000  100% /
devtmpfs                  485832        0 485832    0% /dev
tmpfs                     497960        0 497960    0% /dev/shm
tmpfs                     497960     7780 490180    2% /run
tmpfs                     497960        0 497960    0% /sys/fs/cgroup
/dev/sda1                1038336   135304 903032   14% /boot
tmpfs                      99596        0  99596    0% /run/user/0
[root@service ~]# 
```

**xfs_growfs**

使用xfs_growfs可以将xfs文件系统进行online方式的扩展，它会将data block进行调整。

```bash
[root@service ~]# xfs_growfs /dev/mapper/centos-root 
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=818944 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=3275776, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 3275776 to 4585472
[root@service ~]# 

```

再次确认df状态, 添加的5G空间已经有效，使用率也降到了72%。

```bash
[root@service ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   18G   13G  5.1G   72% /
devtmpfs                 475M     0  475M    0% /dev
tmpfs                    487M     0  487M    0% /dev/shm
tmpfs                    487M  7.6M  479M    2% /run
tmpfs                    487M     0  487M    0% /sys/fs/cgroup
/dev/sda1               1014M  133M  882M   14% /boot
tmpfs                     98M     0   98M    0% /run/user/0
[root@service ~]# 

```

