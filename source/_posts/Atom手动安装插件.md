---
title: Atom手动安装插件
date: 2018-03-02 15:26:13
tags:
- Atom
categories:
- tools
- Atom使用经验
---

项目中需要解析xml文件, 但是客户发过来的文件并没有格式化, 需要安装格式化插件, 直接从官方repo安装不上, 遂有此文, 以作备忘.

1. 获取repo地址
    打开Atom的插件搜索, 搜索需要的插件, 获取对应的源代码地址.
    ![插件](http://p3euxxfa8.bkt.clouddn.com/atomdemo.png)
    上图中箭头所指位置就是源码所在地址.

2. 下载到本地.
    一般扩展插件都会被安装到系统用户目录下的.atom/packages中, 命令行切换到目标目录, 将源代码克隆到本地.
    ```
    git clone https://github.com/Glavin001/atom-beautify.git
    ```

3.安装
    cd到对应的目录, 执行npm install即可安装对应的插件, 一般不出意外就可以安装成功了.
