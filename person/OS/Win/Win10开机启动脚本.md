
# 开关机自启脚本

## 开机自启脚本

输入`shell:startup`，打开用户启动文件夹，将bat启动脚本放到这个文件夹下。

## 关机启动脚本

> 执行`gpedit.msc`，逐级打开“本地计算机策略”-->“计算机配置”-->“Windows设置”-->“脚本”

> PowerShell脚本  
> 开机：`"C:\softwares\VMware\VMware Workstation\vmrun.exe" start "C:\softwares\VMware\VMware Machines\CentOS 7\CentOS 7.vmx" nogui`  
> 关机：`"C:\softwares\VMware\VMware Workstation\vmrun.exe" stop "C:\softwares\VMware\VMware Machines\CentOS 7\CentOS 7.vmx" soft`

## VMware命令行工具

官方参考：<https://docs.vmware.com/en/VMware-Fusion/10.0/com.vmware.fusion.using.doc/GUID-24F54E24-EFB0-4E94-8A07-2AD791F0E497.html>