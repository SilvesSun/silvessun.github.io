---
title: Dockerfile编写的最佳实践
date: 2019-03-05 15:35:46
tags:
- 读书笔记
categories:
- Docker
---

## 通用规则和建议

### 通过 Dockerfile 构建的镜像所启动的容器应该尽可能短暂

### 使用 .dockerignore 文件来提高效率

### 避免安装不需要的依赖

### 每个镜像应该只关注一个问题
将应用解耦有助于横向扩展和镜像复用. 举例来说, 一个web应用应该分离为三个镜像, 应用本身/数据库/缓存

你可能听过"一个容器只运行一个进程", 大部分情况下这是很好的, 但是这并不是必须的. 

如果容器相互依赖, 可以使用容器网络来保证容器间通信

### 最小化容器的层数

### 多行参数排序
将需要安装的包安装字母排序, 有助于避免安装重复的包, 并易于阅读. "\" 前加入空格便于阅读

```
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

### 构建缓存
构建镜像的过程中, Docker 会遍历 Dockerfile 并按顺序执行. 在执行每条命令之前, 都会检查是否有缓存可用而不是重新生成一个新的镜像. 可用使用 `--no-cache=true`
来禁用缓存

- 从一个基础镜像开始（FROM 指令指定），下一条指令将和该基础镜像的所有子镜像进行匹配，检查这些子镜像被创建时使用的指令是否和被检查的指令完全一样。如果不是，则缓存失效
- 多数情况只需要对比Dockerfile 中镜像的指令, 特殊情况需要更详细的标注
- 对于 ADD 和 COPY 指令，镜像中对应文件的内容也会被检查，每个文件都会计算出一个校验和。文件的最后修改时间和最后访问时间不会纳入校验。在缓存的查找过程中，会将这些校验和和已存在镜像中的文件校验和进行对比。如果文件有任何改变，比如内容和元数据，则缓存失效
- 除了 ADD 和 COPY 指令，缓存匹配过程不会查看临时容器中的文件来决定缓存是否匹配。例如，当执行完 RUN apt-get -y update 指令后，容器中一些文件被更新，但 Docker 不会检查这些文件。这种情况下，只有指令字符串本身被用来匹配缓存

## Dockerfile指令使用建议

### FROM 
尽可能使用官方镜像作为基础镜像
### LABEL 
一个镜像可包含多个标签, 建议将多个标签放到一个 LABEL 中
### RUN 
永远将 RUN apt-get update 和 apt-get install 组合成一条 RUN 声明. 

```
RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```

将 apt-get update 放在一条单独的 RUN 声明中会导致缓存问题以及可能后续的 apt-get install 失败.

使用 RUN apt-get update && apt-get install -y 可以确保你的 Dockerfiles 每次安装的都是包的最新的版本，而且这个过程不需要进一步的编码或额外干预。
这项技术叫作 cache busting。你也可以显示指定一个包的版本号来达到 cache-busting，这就是所谓的固定版本

### CMD
CMD 大多数情况下都应该以 CMD ["executable", "param1", "param2"...] 的形式使用. 

### EXPOSE
为应用程序使用常见的端口

### ENV
为了方便新程序运行，你可以使用 ENV 来为容器中安装的程序更新 PATH 环境变量。例如使用 ENV PATH /usr/local/nginx/bin:$PATH 来确保 CMD ["nginx"] 能正确运行

ENV 指令也可用于为你想要容器化的服务提供必要的环境变量，比如 Postgres 需要的 PGDATA

### ADD/COPY
优先使用 COPY, 它比ADD更透明. COPY 只支持简单将本地文件拷贝到容器中, ADD 的最佳用例是将本地 tar 文件自动提取到镜像中 `ADD rootfs.tar.xz`

如果你的 Dockerfile 有多个步骤需要使用上下文中不同的文件。单独 COPY 每个文件，而不是一次性的 COPY 所有文件，这将保证每个步骤的构建缓存只在特定的文件变化时失效

为了让镜像尽量小，最好不要使用 ADD 指令从远程 URL 获取包，而是使用 curl 和 wget. 这样你可以在文件提取完之后删掉不再需要的文件来避免在镜像中额外添加一层

### ENTRYPOINT

ENTRYPOINT 的最佳用处是设置镜像的主命令. ENTRYPOINT 指令也可以结合一个辅助脚本使用

### VOLUME
VOLUME 指令用于暴露任何数据库存储文件，配置文件，或容器创建的文件和目录。强烈建议使用 VOLUME 来管理镜像中的可变部分和用户可以改变的部分

### USER
如果某个服务不需要特权执行，建议使用 USER 指令切换到非 root 用户

你应该避免使用 sudo，因为它不可预期的 TTY 和信号转发行为可能造成的问题比它能解决的问题还多。如果你真的需要和 sudo 类似的功能（例如，以 root 权限初始化某个守护进程，以非 root 权限执行它），你可以使用 gosu。

最后，为了减少层数和复杂度，避免频繁地使用 USER 来回切换用户。

### WORKDIR
使用绝对路径. 使用 WORKDIR 来替代类似于 RUN cd ... && do-something 的指令，后者难以阅读、排错和维护

## 例子

```shell
FROM buildpack-deps:stretch-scm

# gcc for cgo
RUN apt-get update && apt-get install -y --no-install-recommends \
		g++ \
		gcc \
		libc6-dev \
		make \
		pkg-config \
	&& rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.12

RUN set -eux; \
	\
# this "case" statement is generated via "update.sh"
	dpkgArch="$(dpkg --print-architecture)"; \
	case "${dpkgArch##*-}" in \
		amd64) goRelArch='linux-amd64'; goRelSha256='750a07fef8579ae4839458701f4df690e0b20b8bcce33b437e4df89c451b6f13' ;; \
		armhf) goRelArch='linux-armv6l'; goRelSha256='ea0636f055763d309437461b5817452419411eb1f598dc7f35999fae05bcb79a' ;; \
		arm64) goRelArch='linux-arm64'; goRelSha256='b7bf59c2f1ac48eb587817a2a30b02168ecc99635fc19b6e677cce01406e3fac' ;; \
		i386) goRelArch='linux-386'; goRelSha256='3ac1db65a6fa5c13f424b53ee181755429df0c33775733cede1e0d540440fd7b' ;; \
		ppc64el) goRelArch='linux-ppc64le'; goRelSha256='5be21e7035efa4a270802ea04fb104dc7a54e3492641ae44632170b93166fb68' ;; \
		s390x) goRelArch='linux-s390x'; goRelSha256='c0aef360b99ebb4b834db8b5b22777b73a11fa37b382121b24bf587c40603915' ;; \
		*) goRelArch='src'; goRelSha256='09c43d3336743866f2985f566db0520b36f4992aea2b4b2fd9f52f17049e88f2'; \
			echo >&2; echo >&2 "warning: current architecture ($dpkgArch) does not have a corresponding Go binary release; will be building from source"; echo >&2 ;; \
	esac; \
	\
	url="https://golang.org/dl/go${GOLANG_VERSION}.${goRelArch}.tar.gz"; \
	wget -O go.tgz "$url"; \
	echo "${goRelSha256} *go.tgz" | sha256sum -c -; \
	tar -C /usr/local -xzf go.tgz; \
	rm go.tgz; \
	\
	if [ "$goRelArch" = 'src' ]; then \
		echo >&2; \
		echo >&2 'error: UNIMPLEMENTED'; \
		echo >&2 'TODO install golang-any from jessie-backports for GOROOT_BOOTSTRAP (and uninstall after build)'; \
		echo >&2; \
		exit 1; \
	fi; \
	\
	export PATH="/usr/local/go/bin:$PATH"; \
	go version

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH
```
