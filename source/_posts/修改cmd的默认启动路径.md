---
title: 修改cmd的默认启动路径
date: 2018-10-08 15:50:28
tags:
- 实用技巧
categories:
- windwos
---

打开注册表`win+r` > `regedit`

找到 `HKEY_CURRENT_USER\Software\Microsoft\Command Processor`

新建字符串项`autorun`

并修改值为想要的路径`cd /d C:\Users\veahow\Desktop（字符串cd /d之后接空格加上你想修改的默认路径`