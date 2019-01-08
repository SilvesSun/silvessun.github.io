---
title: unknown host问题解决
date: 2018-10-23 09:52:57
tags: 
- DNS解析
categories:
- linux踩坑记录
---

最近两天在服务器安装包时发现DNS解析出了问题. 所有的DNS无法解析, 但是网络能ping通IP地址.

解决办法:

##1 检查 `/etc/resolv.conf`. 但是我的`resolv.conf`解析没有问题.

##2 检查网卡相关配置

`cat /etc/sysconfig/network-scripts/ifcfg-vnet0`

```shell
DEVICE=venet0
BOOTPROTO=static
ONBOOT=yes
ARPCHECK="no"
IPADDR=127.0.0.1
NETMASK=255.255.255.255
BROADCAST=0.0.0.0
ARPCHECK="no"
IPV6INIT="yes"
GATEWAY=192.168.0.1
```

##3 检查dns解析服务
`grep hosts /etc/nsswitch.conf`

```shell
#hosts:     db files nisplus nis dns
hosts:      files dns myhostname
```

我这里就是第三步缺少了dns这个配置选项, 修改后即正常



