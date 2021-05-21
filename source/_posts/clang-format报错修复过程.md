---
title: clang-format报错修复过程
category:
  - linux
tags:
  - linux
  - clang-format
date: 2021-04-27 20:06:57
---

## 1.前言

今天在给vscode适配clang-format的过程中遇到了一个警告，平时用vim时没有被暴露出来，在vscode中每次调用到都会弹出：

```bash
 /usr/lib64>  clang-format
clang-format: /lib64/libtinfo.so.5: no version information available (required by clang-format)
```

今天花了点时间解决，记录下过程~

<!-- more -->

## 2.尝试

首先根据网上查阅到的资料，尝试了几种方法：

### i.删除链接文件

```bash
 /usr/lib64> ls -la libtinfo.so.5*
lrwxrwxrwx 1 root root     15 Apr 27 14:52 libtinfo.so.5 -> libtinfo.so.5.9
-rwxr-xr-x 1 root root 174576 Sep  7  2017 libtinfo.so.5.9
```

libtinfo.so.5本身是libtinfo.so.5.9的链接，按网上方法删掉后，确实不报错了，但是再登录docker的时候，bash就报找不到libtinfo.so.5文件，登录不了bash了。当时又已经退出了docker的登录，进退两难😿。我试着在外面按里面的路径造了一个链接文件，再用docker cp指令传入docker，终于成功登进去了。第一种方法失败了。

### ii.安装最新libtinfo5

网上说可以用以下指令：

```bash
sudo apt update && sudo apt install -y libtinfo5
```

安装最新的libinfo5。不过我们服务器是centos，使用的yum，yum安装中没有找到libtinfo5，这条路也走不通了。

### iii.更新程序

网上看到有其他haskell等程序会报相同的错，更新重装就好了。于是我开始研究怎么更新重装clang-format。但是我发现我服务器上的clang-format是带在llvm中的，yum中也没有单独安装clang-format的选项，llvm太大了，我暂时考虑放弃这条路。

## 3.问题解决

我发现同事的镜像中clang-format没有报这个错，于是拉下来研究，发现他们的clang-format是npm装的：

```bash
 /usr/bin> ls -la clang-format
lrwxrwxrwx 1 root root 41 Apr 27 09:58 clang-format -> ../lib/node_modules/clang-format/index.js
```

而我的是llvm带的，而且莫名其妙有2个：

```bash
/usr/bin> whereis clang-format
clang-format: /usr/local/bin/clang-format /opt/clang+llvm-10.0.1-x86_64-linux-gnu-ubuntu-16.04/bin/clang-format

/usr/bin> ls -la clang-format
lrwxrwxrwx 1 root root 32 Aug 31  2020 clang-format -> /usr/local/llvm/bin/clang-format
```

我重新使用npm安装了clang-format，变成了3个，且执行clang-format还是报错：

```bash
/usr/bin> npm install -g clang-format
......

/usr/bin> whereis clang-format
clang-format: /usr/bin/clang-format /usr/local/bin/clang-format /opt/clang+llvm-10.0.1-x86_64-linux-gnu-ubuntu-16.04/bin/clang-format

/usr/bin> clang-format
clang-format: /lib64/libtinfo.so.5: no version information available (required by clang-format)
```

应该是调用到了原来的clang-format，那这里就有个问题，有多个同名的可执行文件在$PATH中时，会先调用哪个。经过查阅，是根据$PATH的先后顺序来决定的，然后使用which可以查看实际使用的指令位置。可以看到，/usr/local/bin在/usr/bin前面，所以先调用了原先llvm的clang-format。

```bash
/usr/bin> echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

/usr/bin> which clang-format
/usr/local/bin/clang-format
```

正好学习下which和where的区别：

```bash
/usr/bin>  whatis which
which                (1)  - shows the full path of (shell) commands

/usr/bin>  whatis whereis
whereis              (1)  - locate the binary, source, and manual page files for a command
```

大致是：

* which展示的是shell实际调用的指令的位置

* whereis会搜索一个指令所有可能调用的文件

因此需要知道调用的实际文件，需要使用which，会更准确。

于是我把llvm的链接删除后，clang-format终于正常了，大功告成😜。

