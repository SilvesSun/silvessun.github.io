---
title: 解决Windows MySQL-python安装不成功的简单方法
date: 2018-02-05 12:05:22
tags:
- Python扩展
categories:
- Python
---

上周在使用MySQL 的时候, 需要安装mysql-python的驱动. 但是直接
```
pip install MySQL-python
```
会报错.
```
_mysql.c(42) : fatal error C1083: Cannot open include file: 'config-win.h': No such file or dire
ctory
```
谷歌到的相关的解决方法如[csdn](http://blog.csdn.net/xxm524/article/details/48754139)的也无法解决问题. 最后发现
一个很简单的办法.

在[www.lfd.uci.edu](https://www.lfd.uci.edu/~gohlke/pythonlibs/)中搜索MySQL_python, 然后下载对应版本的connect, 然后执行
```
pip install MySQL_python‑1.2.5‑cp27‑none‑win_amd64.whl
```
可以很顺利的安装成功.
