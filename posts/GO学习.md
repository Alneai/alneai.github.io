---
categories: []
date: '2024-06-26'
description: GO相关的简单总结
tags: []
title: GO学习
updated: '2024-06-26T11:42:02.180+08:00'
---
# GO学习

## vim环境配置

1. `~/.bashrc`下添加环境变量

```shell
export GOROOT=/usr/local/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
export GOPATH=~/mygo
```

2. 安装vim插件管理器Vundle

```shell
mkdir ~/.vim/bundle
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

需要的插件：

- Vim-go
- Your Complete Me

5. 编辑`~/.vimrc`配置

```shell
"Vundle 相关配置

" 关闭兼容模式
set nocompatible

set nu " 设置行号
set cursorline "突出显示当前行
" set cursorcolumn " 突出显示当前列
set showmatch " 显示括号匹配

" tab 缩进
set tabstop=4 " 设置Tab长度为4空格
set shiftwidth=4 " 设置自动缩进长度为4空格
set autoindent " 继承前一行的缩进方式，适用于多行注释

" 开启实时搜索
set incsearch
" 搜索时大小写不敏感
set ignorecase
syntax enable
syntax on                    " 开启文件类型侦测
filetype plugin indent on    " 启用自动补全
```
