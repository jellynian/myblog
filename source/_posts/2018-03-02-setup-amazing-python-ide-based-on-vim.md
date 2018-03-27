---
layout: post
title: "使用vim搭建强大的Python IDE"
description: ""
category: 
tags: [vim,python]
last_updated: 2018年 03月 02日 星期五 13:01:44 CST
---

# 安装vim8

ubuntu 16.04 默认的vim为vim7，我们需要较为新的vim8 来支持更多的新特性。你可以使用命令 `vim --version` 来查看你本机的vim 版本。


    sudo add-apt-repository ppa:jonathonf/vim -y
    sudo apt update
    sudo apt install vim


macos 安装则使用brew即可

    brew install vim 

# 安装vim插件管理工具 Vundle

管理vim插件的工具有很多，我曾经使用过的有 [vim-plug](https://github.com/junegunn/vim-plug) 和 Vundle 这里我们推荐[Vundle](https://github.com/VundleVim/Vundle.vim).

安装Vundle


    git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim


在 `~/.vimrc` 文件中写入如下内容,写入完毕后保存退出，后续的文章会告诉大家每一个插件的作用。

    set nocompatible              " be iMproved, required
     
    " set the runtime path to include Vundle and initialize
    set rtp+=~/.vim/bundle/Vundle.vim
    call vundle#begin('~/.vim/bundle/')
    " alternatively, pass a path where Vundle should install plugins
    "call vundle#begin('~/some/path/here')
     
    " let Vundle manage Vundle, required
    Plugin 'VundleVim/Vundle.vim'                                                                                                                                                                               
    Plugin 'vim-airline/vim-airline'
    Plugin 'vim-airline/vim-airline-themes'
    Plugin 'scrooloose/nerdtree'
    Plugin 'w0rp/ale'
    Plugin 'tomasr/molokai'
    Plugin 'altercation/vim-colors-solarized'
    Plugin 'Valloric/YouCompleteMe'
    Plugin 'jnurmine/Zenburn'
    Plugin 'vim-scripts/minibufexplorerpp'
    Plugin 'jistr/vim-nerdtree-tabs'
    Plugin 'kien/ctrlp.vim'
    Plugin 'Chiel92/vim-autoformat'
    Plugin 'tell-k/vim-autopep8'
    Plugin 'jiangmiao/auto-pairs'
    Plugin 'scrooloose/nerdcommenter'
    Plugin 'Yggdroot/indentLine'
    " All of your Plugins must be added before the following line
    call vundle#end()            " required
    filetype plugin indent on    " required


可以看到 `call vundle#end()`  和 `call vundle#begin('~/.vim/bundle/')` 之间的内容是插件列表，这些插件的命名方式其实是 github的用户名和仓库名组成的，在vim中执行  `BundleInstall`
命令，其实是通过git 来克隆这些仓库，来达到安装插件的目的。

![](/img/vim-vundle.png)

# 安装插件 

在vim中执行 `BundleInstall` Vundle插件会调用git下载列表中的插件。
注意 'Valloric/YouCompleteMe' 插件需要下载各种依赖，可能需要花费大量的时间如果下载中断，你可以直接执行,来安装YouCompleteMe


    git clone --recursive https://github.com/Valloric/YouCompleteMe.git  ~/.vim/bundle/YouCompleteMe
    git submodule update --init --recursive   # 检查git仓库完整性

# 编译YouCompleteMe


    cd ~/.vim/bundle/YouCompleteMe
    sudo apt install build-essential cmake
    ./install.py  # 如果你需要c/c++的自动提示支持，可以添加参数 --clang-completer


# 插件介绍

    Plugin 'vim-airline/vim-airline'
    Plugin 'vim-airline/vim-airline-themes'
    Plugin 'scrooloose/nerdtree'
    Plugin 'w0rp/ale'
    Plugin 'tomasr/molokai'
    Plugin 'altercation/vim-colors-solarized'
    Plugin 'Valloric/YouCompleteMe'
    Plugin 'jnurmine/Zenburn'
    Plugin 'vim-scripts/minibufexplorerpp'
    Plugin 'jistr/vim-nerdtree-tabs'
    Plugin 'kien/ctrlp.vim'
    Plugin 'Chiel92/vim-autoformat'
    Plugin 'tell-k/vim-autopep8'
    Plugin 'jiangmiao/auto-pairs'
    Plugin 'scrooloose/nerdcommenter'
    Plugin 'Yggdroot/indentLine'

## vim-airline
这是一款漂亮底部提示条插件，拥有着漂亮的外观

## scrooloose/nerdtree

这是一款好用的文件列表插件

## ale
这是一款语法检查插件

