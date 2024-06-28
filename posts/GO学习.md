---
categories: []
date: '2024-06-26'
description: GO相关的简单总结
tags:
- Go
title: GO学习
updated: '2024-06-28T15:48:03.194+08:00'
---
# GO学习

## 安装与vim环境配置

1. wegt下载go，`~/.bashrc`下添加环境变量

```shell
export GOROOT=/usr/local/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
export GOPATH=~/mygo
```

`source .bashrc`刷新配置

2. 安装vim插件管理器Vundle

```shell
mkdir ~/.vim/bundle
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

3. 编辑 `~/.vimrc`配置

```vim
 "Vundle 相关配置
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'
Plugin 'fatih/vim-go'
Plugin 'Valloric/YouCompleteMe'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required

" 关闭兼容模式
set nocompatible

set nu " 设置行号
set showmatch " 显示括号匹配

" tab 缩进
set tabstop=4 " 设置Tab长度为4空格
set shiftwidth=4 " 设置自动缩进长度为4空格
set autoindent " 继承前一行的缩进方式，适用于多行注释

set incsearch                " 开启实时搜索
set ignorecase               " 搜索时大小写不敏感
syntax enable
syntax on                    " 开启文件类型侦测
filetype plugin indent on    " 启用自动补全
```

4. 安装和配置插件

- vim-go
- Your Complete Me

在Vim内执行 `:PluginInstall`安装插件，执行`:GoInstallBinaries`安装vim-go所需环境

## 常用命令

- go build -o
- go get，获取远程包
- go install
- go run，编译并运行
- go env，查看go环境变量
- go list，查看安装的包
- go mod init example.com/greetings，初始化模块

## GMP调度机制

* G - Goroutine，Go协程，是参与调度与执行的最小单位
* M - Machine，指的是系统级线程，与内核调度实体KSE关联
* P - Processor，指的是逻辑处理器，P关联了的本地可运行G的队列(也称为LRQ)，最多可存放256个G。最大值为`runtime.GOMAXPROCS()`。

调度流程：

* 线程M想运行任务就需得获取 P，即与P关联。
* 然从 P 的本地队列(LRQ)获取 G
* 若LRQ中没有可运行的G，M 会尝试从全局队列(GRQ)拿一批G放到P的本地队列，
* 若全局队列也未找到可运行的G时候，M会随机从其他 P 的本地队列偷一半放到自己 P 的本地队列。
* 拿到可运行的G之后，M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。

## 内存管理

### GC

**三色标记清除算法**，由于支持写屏障，GC过程和程序可以并发运行

- 黑色集合：没有引用任何白色对象
- 白色集合：要回收的对象，可以指向黑色对象
- 灰色集合：可能指向白色对象，中间状态

当垃圾回收开始，全部对象标记为白色，然后垃圾回收器会遍历所有根对象并把它们标记为灰色。**根对象**就是程序能直接访问到的对象，包括全局变量以及栈、寄存器上的里面的变量。在这之后，垃圾回收器选取一个灰色的对象，首先把它变为黑色，然后开始寻找去确定这个对象是否有指针指向白色集合的对象，若找到则把找到的对象由标记为灰色，并将其白色集合中移入到灰色集合中。就这样持续下去，直到灰色集合中没有任何对象为止。

每次堆中的指针被修改时候写屏障都会执行，写屏障会将该指针指向的对象标记为灰色，然后放入灰色集合，然后继续扫描该对象。
