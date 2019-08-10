---
title: play框架学习(应用结构)
date: 2018-08-17 14:22:18
tags:
- scala
- play
categories:
- play2
---

![2018-08-17-14-45-31](http://p3euxxfa8.bkt.clouddn.com/2018-08-17-14-45-31.png)

执行sbt new applicationName 后生成的文件结构如下

```
app                      → Application sources
 └ assets                → Compiled asset sources
    └ stylesheets        → Typically LESS CSS sources
    └ javascripts        → Typically CoffeeScript sources
 └ controllers           → Application controllers
 └ models                → Application business layer
 └ views                 → Templates
build.sbt                → Application build script
conf                     → Configurations files and other non-compiled resources (on classpath)
 └ application.conf      → Main configuration file
 └ routes                → Routes definition
dist                     → Arbitrary files to be included in your projects distribution
public                   → Public assets
 └ stylesheets           → CSS files
 └ javascripts           → Javascript files
 └ images                → Image files
project                  → sbt configuration files
 └ build.properties      → Marker for sbt project
 └ plugins.sbt           → sbt plugins including the declaration for Play itself
lib                      → Unmanaged libraries dependencies
logs                     → Logs folder
 └ application.log       → Default log file
target                   → Generated stuff
 └ resolution-cache      → Info about dependencies
 └ scala-2.11
    └ api                → Generated API docs
    └ classes            → Compiled class files
    └ routes             → Sources generated from routes
    └ twirl              → Sources generated from templates
 └ universal             → Application packaging
 └ web                   → Compiled web assets
test                     → source folder for unit or functional tests
```

## app
app目录是项目的源码位置, 包括 controllers, models, views 三个文件夹, 分别对应 MVC 架构的三个模块. 在实际的项目开发中, 也可以手动添加其他的模块

## public
public 文件夹用来存放直接由 web server 提供的文件, 例如 images, CSS, js 文件. 在新建应用中, public 对应的 url 映射是 /assets

## conf
conf 文件夹主要是项目配置信息, 主要包括两个配置文件: application.conf 和 routes. application.conf 定义了绝大部分参数值, routes 定义了路有关系

## lib
可选的 lib 文件夹用来放置额外的依赖

## project
定义 sbt 构建选项

- plugins.sbt 定义了 sbt 的插件

    ```
    [plugins.sbt ]

    addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.6.17")
    ```

- build.properties 定义了 sbt 版本

## target
应用编译完成后的所有文件

