---
categories: []
date: '2024-06-26'
description: GO相关的简单总结
tags:
- Go
title: GO学习
updated: '2024-06-28T09:37:37.436+08:00'
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

## 语法

### 常用命令

- go build -o
- go get，获取远程包
- go install
- go run，编译并运行
- go env，查看go环境变量
- go list，查看安装的包
- go mod init example.com/greetings，初始化模块
