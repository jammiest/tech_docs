# 本地服务器静态IP设置

## 虚拟机网络配置
> 对虚拟机网络适配器设置为桥接模式，不勾选复制物理网络状态

## 配置文件

> /etc/sysconfig/network-scripts/ifcfg-*** 【对应iconfig命令行中的设备号,如ifcfg-ens33】

```ini
ONBOOT=yes //是否启用该设备
BOOTPROTO=none //手动(none/static)还是自动(dhcp)
IPADDR=192.168.10.244 //固定IP地址
PREFIX=24	//掩码
GATEWAY=192.168.10.1	//网关
DNS1=10.0.2.31	//DNS (备用：10.0.2.81，114.114.114.114)
;UUID=e96a02cf-db79-46ca-bbf3-6954542d75db //注释uuid
```

## 重启network服务

```bash
/etc/init.d/network restart
service network restart
```

## ping命令检测网络状态
```bash
ping www.baidu.com
```
