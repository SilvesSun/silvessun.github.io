---
title: centos安装Python3
date: 2019-01-18 09:38:58
tags:
- python3安装
- yum源配置
categories:
- Linux配置
---

![](http://supcoder.net/centos7.jpg)

## yum源更新
除了os类的yum源，还需要配置几个常用的源：epel、ius

`EPEL`是专门为RHEL、CentOS等Linux发行版提供额外rpm包的。很多os中没有或比较旧的rpm，在epel仓库中可以找到
`IUS`只为RHEL和CentOS这两个发行版提供较新版本的rpm包

### 安装yum源
`IUS`源依赖`epel`源, 首先安装`epel`源, 再安装`ius`源

`yum -y install epel-release`

`rpm -ivh https://centos7.iuscommunity.org/ius-release.rpm  # CentOS 7`

### 注意事项
安装epel源后, 有时候epel在yum源的禁用列表中, 需要修改配置文件. 将epel源配置文件中的`enabled`从0改为1
```shell
vim /etc/yum.repos.d/epel.repo
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1 # 修改此处
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
#baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch/debug
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/7/SRPMS
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=1  # 修改此处
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
```

## 安装python3
- 1.依赖安装
```shell
yum install wget gcc make # 下载,编译
yum install zlib-devel  # 内建模块 zipimport依赖
yum install readline-devel # 解决python中方向键等乱码问题
yum install  bzip2-devel # 解决 import bz2 报错
yum install  ncurses-devel # 解决 import curses 报错
```

- 2.1源码安装
[官网](https://www.python.org/downloads/)

下载解压缩后进入文件目录
```shell
cd Python-3.6.1
./configure --prefix=/usr/local/python3.6 --enable-optimizations
make && make install 
```

- 2.2 使用ius源安装最新的python
![yum安装python36](http://supcoder.net/yum安装python36.png)



