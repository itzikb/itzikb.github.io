---
layout: single
permalink: /installing-vim-plugins-using-vundle/
title: "Installing Vim Plugins using Vundle"
tags: vim
comments: true
---
 **Vim&nbsp;** is a highly configureable text editor. One nice thing about Vim is that it has lots of plugins to extend it's functionality from Color schema to using Git etc.&nbsp;  
**Vundle** is a great tool for managing Plugins for Vim with ease.

### Installing Vim
I'm using dnf to install the packages because I use Fedora. Replace it with yum if you are using RHEL/CentOS.
```shell
$ sudo dnf -y install vim
```
You may encounter problems if vim-minimal is installed:
```shell
$ sudo dnf -y install vim
Fedora 23 - x86_64 - Updates 1.8 MB/s | 25 MB 00:13
Jenkins 125 kB/s | 54 kB 00:00
Dependencies resolved.
================================================================================
Package Arch Version Repository Size
================================================================================
Installing:
gpm-libs x86_64 1.20.7-7.fc23 fedora 36 k
vim-common x86_64 2:7.4.1868-1.fc23 updates 6.3 M
vim-enhanced x86_64 2:7.4.1868-1.fc23 updates 1.2 M
vim-filesystem x86_64 2:7.4.1868-1.fc23 updates 29 k

Transaction Summary
================================================================================
Install 4 Packages

Total download size: 7.6 M
Installed size: 29 M
Downloading Packages:
(1/4): gpm-libs-1.20.7-7.fc23.x86_64.rpm 82 kB/s | 36 kB 00:00
(2/4): vim-filesystem-7.4.1868-1.fc23.x86_64.rp 88 kB/s | 29 kB 00:00
(3/4): vim-enhanced-7.4.1868-1.fc23.x86_64.rpm 671 kB/s | 1.2 MB 00:01
(4/4): vim-common-7.4.1868-1.fc23.x86_64.rpm 563 kB/s | 6.3 MB 00:11
--------------------------------------------------------------------------------
Total 557 kB/s | 7.6 MB 00:13
Running transaction check
Transaction check succeeded.
Running transaction test
The downloaded packages were saved in cache until the next successful transaction.
You can remove cached packages by executing 'dnf clean packages'.
Error: Transaction check error:
file /usr/share/man/man1/vim.1.gz from install of vim-common-2:7.4.1868-1.fc23.x86_64 conflicts with file from package vim-minimal-2:7.4.827-1.fc23.x86_64

Error Summary
-------------
```
You'll have to remove vim-minimal and install vim. Note that it will also remove sudo so you'll need to do the following as root or by running sudo -s.
```shell
$ dnf -y remove vim-minimal;dnf -y install vim sudo
```
### Installing Vundle
Install git If you don't have it
```shell
$ sudo dnf -y install git
```
Clone the Vundle git repository
```shell
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
Put this at the top of your ~/.vimrc file
```shell
filetype off " required
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()                
Plugin 'VundleVim/Vundle.vim'          
Plugin 'baskerville/bubblegum       
call vundle#end() " required       
filetype plugin indent on
colorscheme bubblegum-256-dark
```

**Note:** Only the Vundle Plugin is required. The other plugins is just for illustration. This applies also to the colorscheme. Install the Plugins by running
```shell
$ vim +PluginInstall +qall
```
To see which plugins installed you can

```shell
$ vim +PluginList
```

It will open the editor with content similar to:
```
" My Plugins
Plugin 'VundleVim/Vundle.vim'
Plugin 'baskerville/bubblegum'
```
To get Vundle help you can type the following inside the editor
```vim
:h vundle
```
