---
title: go包管理
date: 2018-06-12 12:00:47
tags:
- gopath
- 包管理
categories:
- go学习
---

## GOPATH
GOPATH的作用是告诉Go 命令和其他相关工具，在那里去找到安装在你系统上的Go包。 所以所有Golang安装包以外的代码, 都需要放置在gopath下. 在win上, 默认的gopath是用户目录下的go文件夹. 可通过 `go env` 查看
![查看gopath](http://p3euxxfa8.bkt.clouddn.com/2018-06-12-12-09-10.png)

如果设置多个gopath目录, 那么go程序会按照配置的顺序依次去寻找有没有对应的依赖包.

这自然而然的引出一个问题, 对于不同的项目, 并不想共享同一个gopath, 以免造成环境污染. 解决这个问题的方法就是针对不同的项目设置不同的gopath, 这就是最原始的go包管理机制.

## vendor
vendor机制就是在项目中添加一个vendor文件夹, go默认它为一个gopath. 这样就可以将依赖库放到这里了

## govendor
govendor将源码拷贝到vendor目录下, 通过在vendor目录维护一个vendor.json的文件, 指定使用的文件. 