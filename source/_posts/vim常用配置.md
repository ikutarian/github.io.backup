---
title: vim常用配置
date: 2019-02-12 09:48:13
tags:
  - vim
  - vimrc
  - nginx.conf
  - 语法高亮
categories:
  - 安装与配置
---

# Vim配置文件的位置

Linux 系统中，输入 `vi`，进入 vim 环境，然后在命令模式下输入 `:version`，拉到下面会看到

```
   system vimrc file: "$VIM/vimrc"
     user vimrc file: "$HOME/.vimrc"
 2nd user vimrc file: "~/.vim/vimrc"
      user exrc file: "$HOME/.exrc"
  fall-back for $VIM: "/usr/share/vim"
```

可以看到系统的 vimrc 文件位置在 `$VIM/vimrc`，用户的 vimrc 文件在 `$HOME/.vimrc`

<!-- more -->

`$VIM` 一般是 `/etc/vim`，`$HOME` 是用户的目录，一般是 `/用户名` 这个文件夹

# 修改配置

进入 `$HOME` 目录

```
cd ~
```

新建一个 `.vimrc` 文件，加入以下内容

```
" tab转空格，并且一个tab等于4个空格
set ts=4
set expandtab
set autoindent

" 显示行号
set number
```

# nginx.conf的语法高亮

默认情况下，vim 对 Nginx 的配置文件是没有高亮的。vim 的官网提供了一份 [nginx.vim](https://www.vim.org/scripts/script.php?script_id=1886)，按照官网的说明使用即可

下载 nginx.vim 文件，保存到 `~/.vim/syntax/`

```
cd ~/.vim/syntax/
wget https://www.vim.org/scripts/download_script.php?src_id=19394 -O nginx.vim
```

然后在 `~/.vim/filetype.vim` 后面加上一行，文件如果不存在就创建一个

```
au BufRead,BufNewFile /etc/nginx/*,/usr/local/nginx/conf/* if &ft == '' | setfiletype nginx | endif 
```