---
title: Linux VMWare Device eth0 does not seem to be present,delaying initialization
date: 2018-01-11 11:55:40
tags:
- Linux
- VMWare
- 网络配置
category:
- Linux
---
虚拟机Vmware上克隆了一个CentOS Linx启动时发现找不到网卡，在命令窗口启动网络服务就会遇到”Device eth0 does not seem to be present, delaying initialization“错误。

### 错误原因：

克隆的Linux系统在新的机器上运行，新服务器网卡物理地址已经改变。而/etc/udev/rules.d/70-persistent-net.rules这个文件确定了网卡和MAC地址的信息之间的绑定，克隆后的网卡的MAC已经发生了变化，所以导致系统认为网络设备不存在，网络不能正常启动。另外一个就是/etc/sysconfig/network-scripts/ifcfg-eth0里面MAC地址也是以前的旧信息。

关于/etc/udev/rules.d/70-persistent-net.rules这个文件，系统在启动时会自动监测变化，然后由/lib/udev/write_net_rules写入到/etc/udev/rules.d/70-persistent-net.rules中一个新的配置节，网卡的的序号依次递增（如原来为eth0,则修改第一后生成一个eth1,再次修改后生成一个eth2...）,且其ATTR{address}的值为当前网卡对应的mac地址。 

### 解决方法：

1.编辑/etc/sysconfig/network-scripts/ifcfg-eth0配置文件，将ifcfg-eth0的配置文件里里面以前的关于MAC地址这一行删除掉或修改。另外克隆的服务器的IP设置的是静态IP，要么修改为一个其它的IP地址或设置为动态IP，重启网卡服务。

2.找到/etc/udev/rules.d/70-persistent-net.rules删除后重启机器，系统会自动生成一个70-persistent-net.rules文件。因为这个文件绑定了网卡和MAC地址，换了网卡以后MAC地址变了，所以不能正常启动，也可以直接编辑这个配置文件把里面的网卡和MAC地址修改成对应的，不过这样多麻烦，直接删除重启，它会自动生成个一个新的文件。 