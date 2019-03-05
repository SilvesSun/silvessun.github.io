---
title: centos7配置service
date: 2019-01-18 10:13:45
tags:
- 自定义service

categories:
- Linux配置
---
![](http://supcoder.net/centos7.jpg)

在`centos6`中, 一般启动一个服务常用`service`命令, 例如`service mysqld start`. 但是在`centos7`中, 使用同样的命令最终执行的并不是`/etc/init.d`目录
下的脚本, 而是使用`/bin/systemctl`去`/usr/lib/systemd/`目录下执行相应的服务. Centos7 开机第一程序从init换成了systemd的启动方式，而systemd依靠unit的方式来控制开机服务，开机级别等功能。

Centos7的服务分为user和system

- /usr/lib/systemd/system   # 系统服务，开机不需要登录就能运行的程序（相当于开机自启）
- /usr/lib/systemd/user     # 用户服务，需要登录后才能运行的程序

目录下又存在两种类型的文件：

- *.service   # 服务unit文件
- *.target    # 开机级别unit

每一个服务以.service结尾，一般会分为3部分：[Unit]、[Service]和[Install]

以mysql的服务脚本为例, 大致说明每个字段的含义.

```shell
#
# Simple MySQL systemd service file
#
# systemd supports lots of fancy features, look here (and linked docs) for a full list: 
#   http://www.freedesktop.org/software/systemd/man/systemd.exec.html
#
# Note: this file ( /usr/lib/systemd/system/mysql.service )
# will be overwritten on package upgrade, please copy the file to 
#
#  /etc/systemd/system/mysql.service 
#  
# to make needed changes.
# 
# systemd-delta can be used to check differences between the two mysql.service files.
#

[Unit]                                       # 服务说明
Description=MySQL Community Server           # 服务描述
After=network.target                         # 描述服务类别，表示本服务需要在network服务启动后在启动
After=syslog.target

[Install]
WantedBy=multi-user.target
Alias=mysql.service

[Service]                                    # 核心区域
User=mysql                                   # 设置服务运行的用户
Group=mysql                                  # 设置服务运行的用户组

# Execute pre and post scripts as root
PermissionsStartOnly=true

# Needed to create system tables etc.
ExecStartPre=/usr/bin/mysql-systemd-start pre

# Start main service
ExecStart=/usr/bin/mysqld_safe --basedir=/usr # 启动服务

# Don't signal startup success before a ping works
ExecStartPost=/usr/bin/mysql-systemd-start post

# Give up if ping don't get an answer
TimeoutSec=600

Restart=always
PrivateTmp=false
```

## 常用字段说明
### Type
    表示后台运行模式。
    simple(默认）：# 以ExecStart字段启动的进程为主进程
    forking:  # ExecStart字段以fork()方式启动，此时父进程将退出，子进程将成为主进程（后台运行）。一般都设置为forking
    oneshot:  # 类似于simple，但只执行一次，systemd会等它执行完，才启动其他服务
    dbus：    # 类似于simple, 但会等待D-Bus信号后启动
    notify:   # 类似于simple, 启动结束后会发出通知信号，然后systemd再启动其他服务
    idle：    # 类似于simple，但是要等到其他任务都执行完，才会启动该服务。

### Restart的类型
    no(默认值)：  # 退出后无操作
    on-success:  # 只有正常退出时（退出状态码为0）,才会重启
    on-failure:  # 非正常退出时，重启，包括被信号终止和超时等
    on-abnormal: # 只有被信号终止或超时，才会重启
    on-abort:    # 只有在收到没有捕捉到的信号终止时，才会重启
    on-watchdog: # 超时退出时，才会重启
    always:      # 不管什么退出原因，都会重启

### RestartSec字段：
    表示systemd重启服务之前，需要等待的秒数：RestartSec: 30

### Exec*字段
    ExecStart：    # 启动服务时执行的命令
    ExecReload：   # 重启服务时执行的命令 
    ExecStop：     # 停止服务时执行的命令 
    ExecStartPre： # 启动服务前执行的命令 
    ExecStartPost：# 启动服务后执行的命令 
    ExecStopPost： # 停止服务后执行的命令

    