---
title: Dockerfile指南
date: 2019-03-01 15:08:39
tags:
- 读书笔记
categories:
- Docker
---

![](http://supcoder.net/Docker_logo.png)

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令

## build

`docker build` 可以通过 `Dockerfile` 和`context` 构建镜像. 这里的`context`指的是特定`PATH`和`URL`中的Dockerfile的集合. PATH是本地系统的一个目录,
URL是一个git地址

build 是由Docker daemon 运行的, 而不是 CLI. 当运行 docker build 的时候, docker 进程首先将指定 context 的所有文件发送到 daemon. 因此, 不要使用根
目录作为 PATH, 这会将宿主机的所有文件导入到 daemon 中.

一般来说, Dockerfile 位于 context 的根. 可以使用 -f 指定特定位置的 Dockerfile; -t 指定生成的镜像保存位置

```cmd
docker build -f /path/to/a/Dockerfile .
docker build -t shykes/myapp .
```

也可以一次指定多个 repo 和 tag

```
docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
```

在构建镜像之前, Docker deamon 会首先检查 Dockerfile 是否有语法错误. 需要注意的是 Dockerfile 中的每一步构建都是相互独立的, 也就是说类似`RUN cd/tmp`
这样的命令对下一步构建不会产生任何作用

为了加快构建速度, Docker 会尽可能的复用一些缓存镜像. 

## Parser directives
Parser directives 是可选的, 用于提示解释器做特殊的处理. 如果需要定义 Parser directives, 需要在 Dockerfile 的第一行定义. 当前, Parser directives
只支持转义字符. 默认的在 Unix-like 中转义字符为`\`, 如果是在 Windows 系统中, 需要修改, 推荐为 

```shell
# escape=`
```

## 申明环境变量(Environment replacement)

在 Dockerfile 中可以通过 `ENV` 申明一个环境变量, 然后在 Dockerfile 中使用 `${variable_name}`

在同一个指令中, 变量共享同一个值. 所以在下面的例子中

```
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```

`def` 的值为`hello`, 而`ghi` 的值为`bye`

## FROM

`FROM` 初始化一层镜像, 并将其设置为接下来的构建的 `Base Image`. 因此, 一个 Dockerfile 的第一句可执行的指令一般都是是 FROM.

```shell
FROM <image> [AS <name>]
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]
```

- `ARG` 是唯一有可能先于 FROM 的指令
- 在一个 Dockerfile 中, FROM 可以多次出现, 每个 FROM 都会清除之前构建命令的所有状态
- 可以通过 AS 对一个镜像指定别名, 这个别名可以在接下来的`FROM` `CPOY`命令中使用
- 可选的 `TAG` / `digest`

## RUN
RUN 会在当前镜像的顶层执行任何命令，并commit成新的（中间）镜像，提交的镜像会在后面继续用到. 两种格式

```shell
RUN <command>：shell格式
RUN ["executable", "param1", "param2"]：exec格式

```

**exec** 格式的 RUN 命令将命令解析为 JSON 格式, 所以应该使用双引号

## CMD 
一个 Dockerfile 中只能有一个 CMD 命令, 多个 CMD 命令只有最后一个生效. **用来指定容器启动时运行的命令。**

```
CMD ["executable", "param1", "param2"]：exec格式
CMD command param1 param2：shell格式
CMD ["param1", "param2"]：省略可执行文件的exec格式，这种写法使CMD中的参数当做ENTRYPOINT的默认参数
```

## LBAEL
给构建的镜像打标签

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...

```

## EXPOSE
为构建的镜像设置监听端口，使容器在运行时监听

## ENV
设置环境变量, 供后续的指令使用. 可以在 RUN 时替换镜像中设置的环境变量 `docker run --env <key>=<value>`

## ADD
复制上下文中的文件到目标镜像内

```
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

- src 文件、目录，也可以是文件URL. 可以指定多个<src>，必须是在上下文目录和子目录中
- 可以是绝对路径，也可以是相对WORKDIR目录的相对路径。所有文件的UID和GID都是0

## ENTRYPOINT
```
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
ENTRYPOINT command param1 param2 (shell form)
```

指定镜像的执行程序，只有最后一条ENTRYPOINT指令有效

## VOLUME
指定镜像内的目录为数据卷

## USER

为接下来的Dockerfile 指令指定用户

## WORKDIR 
## ARG

指定了用户在docker build --build-arg <varname>=<value>时可以使用的参数

