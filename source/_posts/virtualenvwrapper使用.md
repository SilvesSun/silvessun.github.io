---
title: virtualenvwrapper使用
date: 2018-02-02 14:58:02
tags:
- 虚拟环境
- 包管理
categories:
- Python
- tools
---

virtualenvwrapper是一个用来管理Python虚拟环境的virtualenv的扩展管理包, 它将所有的虚拟环境整合到用户的根目录下的ENS文件夹中(以Windows)为例. 从而可以方便的实现虚拟环境的切换, 删除, 复制, 安装虚拟环境.

## 安装
在Windows下, 安装命令为
```
pip install virtualenvwrapper-win
```

这样就能在命令行中直接使用virtualenvwrapper的相关命令了.

## 新建虚拟环境
```
mkvirtualenv testvir
```

在新建虚拟环境的时候可以使用-p指定使用的Python版本

## 常用命令
```
创建基本环境：mkvirtualenv [环境名]

删除环境：rmvirtualenv [环境名]

激活环境：workon [环境名]

退出环境：deactivate

列出所有环境：workon 或者 lsvirtualenv -b
```
