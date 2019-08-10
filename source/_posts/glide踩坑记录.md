---
title: glide踩坑记录
date: 2018-11-21 09:25:30
tags: 
- glide
categories:
- golang
---
![go图标](http://p3euxxfa8.bkt.clouddn.com//18-6-26/73283110.jpg)

这里作为golang glide使用过程中遇到的问题及解决办法的记录. 不定期更新

1. `Error looking for golang.org/x/sys/unix: Cannot detect VCS`

由于众所周知的原因, 国内无法直接获取golang.org 相关的包. 解决办法如下:
- 设置mirror

`glide mirror set https://golang.org/x/sys https://github.com/golang/sys --vcs git`

将golang官网相关的包使用 GitHub 的镜像地址来下载.

- 设置repo

在 `glide.yaml` 中, 一个 `package` 相关的记录通常包括: package, version, repo, vcs. 其中只有package 是必须的. 当在官网无法下载对应的包时, 可以指定
repo 来达到下载的目的

```
- package: golang.org/x/sys
  repo: git@github.com:golang/sys.git
```
![](http://blogsk.oss-us-west-1.aliyuncs.com/golang.png)