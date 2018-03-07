---
title: centos7安装YouCompleteMe
date: 2018-03-07 10:02:13
tags:
- YouCompleteMe安装
- CMake安装
- Vundle安装
- vim安装
categories:
- Linux配置
---

这几天重新撸了一个VPS, 所有的环境都要重新配置一遍, 把配置过程记录下来以作备忘. 操作系统为centos7 64位

## vim安装
首先要将vim升级到最新版本.安装过程如下:
- 更新yum
```
yum upgrade
yum update
```

- 安装git
```
yum install git
```
- 升级gcc
```
yum install centos-release-scl -y
yum install devtoolset-3-toolchain -y
yum install gcc-c++
scl enable devtoolset-3 bash
```
- 安装最新版vim
```
yum install ncurses-devel
wget https://github.com/vim/vim/archive/master.zip
unzip master.zip
cd vim-master/src
./configure --with-features=huge -enable-pythoninterp=yes --with-python-config-dir=/usr/lib/python2.7/config
make && make install
export PATH=/usr/local/bin:$PATH
```

**这里-enable-pythoninterp=yes --with-python-config-dir=/usr/lib/python2.7/config 是为了开启Python支持, 同时后面的路径不同的机器位置可能不一样, 找到对应的config文件夹即可**
- 检查
```
vim --version | grep python
```
输出
![vim检查](http://p3euxxfa8.bkt.clouddn.com/45a3997b9a6e7476f4b06db129fbcf9f.png)

可以看到vim对Python的支持信息

## Vundle安装
vundle（https://github.com/VundleVim/Vundle.vim ) 是一款方便管理vim插件的插件. 可以直接通过以下命令安装Vundle
```shell
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

然后需要在.vimrc中添加相关的配置信息
```
"===================================================================
" vundle 环境设置
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
" vundle 管理的插件列表必须位于 vundle#begin() 和 vundle#end() 之间
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'altercation/vim-colors-solarized'
Plugin 'tomasr/molokai'
Plugin 'vim-scripts/phd'
Plugin 'majutsushi/tagbar'  "基于标签的标识符列表插件
Plugin 'Valloric/YouCompleteMe'  " 补全插件
Plugin 'Lokaltog/vim-powerline'  "状态栏美化
Plugin 'scrooloose/nerdtree'
Plugin 'Yggdroot/indentLine'
Plugin 'jiangmiao/auto-pairs'  "括号和引号自动匹配
Plugin 'tell-k/vim-autopep8'
Plugin 'scrooloose/nerdcommenter'
Plugin 'terryma/vim-expand-region'
Plugin 'fholgado/minibufexpl.vim' "快速浏览和操作buffer
Plugin 'jistr/vim-nerdtree-tabs'
Plugin 'kien/ctrlp.vim'  "super search. key-map: <C-p>
" Zen coding
"Plugin 'mattn/emmet-vim'
"Plugin 'davidhalter/jedi-vim'
Plugin 'ternjs/tern_for_vim'
Plugin 'plasticboy/vim-markdown'
Plugin 'iamcco/markdown-preview.vim'
" 插件列表结束
call vundle#end()
filetype plugin indent on
```

其中Plugin 'tomasr/molokai'这样的就对应一个插件.然后在vim中执行
```shell
:PluginInstall
```
即可开始安装需要的插件

## CMake安装
这样直接安装完成的vim打开会报错
```
The ycmd server SHUT DOWN (restart with ':YcmRestartServer'). YCM core library not detected; you need to compile YCM before using it. Follow the instructions in the documentation
```

打开对应的log文件查看:
```
Traceback (most recent call last):
  File ".ycm_extra_conf.py", line 32, in <module>
    import ycm_core
ImportError: No module named ycm_core
```

解决这个问题的方法在[YouCompleteMe issue](https://github.com/Valloric/YouCompleteMe/issues/1707)中有提到.
```
I use this command can fix this bug
cd ~/.vim/bundle/YouCompleteMe
./install.py --clang-completer
```

这里发现需要安装CMake
### 开始安装CMake
- 获取CMake源码
```
wget https://cmake.org/files/v3.11/cmake-3.11.0-rc2.tar.gz
```

- 解压
```
tar xzvf cmake-3.11.0-rc2.tar.gz
```
- 进入文件夹
```
cd cmake-3.11.0
./bootstrap  
```
- 然后执行gmake
```
gmake
```
- 安装
```
make install
```

这样CMake就成功安装了, 查看
![CMake 检查](http://p3euxxfa8.bkt.clouddn.com/204e15098ef45d149939d65fbbec9b1b.png)
