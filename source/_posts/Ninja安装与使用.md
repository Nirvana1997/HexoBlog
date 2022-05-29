---
title: Ninja安装与使用
category:
  - c++
tags:
  - c++
  - Ninja
  - complie
date: 2022-04-17 14:37:16
---

最近想试试使用Ninja编译，毕竟Ninja是以编译速度为主要目标的构建系统，下面记录下安装使用的过程。
<!-- more -->

## 1.安装

首先将ninja的仓库的release分支下载或者clone下来：

```bash
$ git clone https://github.com/ninja-build/ninja.git && cd ninja
$ git checkout release
```

官方文档这里地址协议使用了git://github.com/ninja-build/ninja.git，但现在使用git会报错：

```bash
$ git clone git://github.com/ninja-build/ninja.git && cd ninja
Cloning into 'ninja'...
fatal: remote error:
  The unauthenticated git protocol on port 9418 is no longer supported.
```

这是最近github取消了git协议，详情参见[Improving Git protocol security on GitHub](https://github.blog/2021-09-01-improving-git-protocol-security-github/)。

然后可以使用python脚本或者cmake去构建可执行程序：

```bash
python:
$ ./configure.py --bootstrap

cmake:
$ cmake -Bbuild-cmake
$ cmake --build build-cmake
```

我这次使用的是python去构建的，然后遇到re2v相关的报错：

``` bash
warning: A compatible version of re2c (>= 0.11.3) was not found; changes to src/*.in.cc will not affect your build.
```

这是缺少了re2c，使用yum或者apt-get安装完就没了，然后在同目录会生成一个ninja的可执行文件，拷贝至`$PATH`对应目录中即可：

```bash
$ ninja --version
1.10.2
```

安装完成后可以使用目录下的ninja_test测试一下：

```bash
$ build-cmake/ninja_test
[343/343] ElideMiddle.ElideInTheMiddle
passed
```

然后根据官方文档，可以使用misc文件夹下的各种文件，实现bash、zsh的补全等功能，使用方法在对应文件的注释中，不过目前感觉没啥补全的需要，暂时没研究：

![misc](misc.png)

## 2.使用

ninja在编译时会去寻找`.ninja`的编译配置文件，默认是`build.ninja`，配置文件的格式大致如下：

```
cflags = -Wall

rule cc
  command = gcc $cflags -c $in -o $out

build foo.o: cc foo.c
```

假如项目本身使用cmake，写好了CMakeList，可以直接使用`cmake -G Ninja`生成build.ninja文件，然后就可以使用ninja指令编译啦~

## 3.编译结果

最后我分别使用cmake和ninja在24核的方式下进行了一次完整编译我们项目，结果如下：

![cmake](cmake.png)

<center>cmake</center>

![ninja](ninja.png)

<center>ninja</center>

ninja大致快了1min，10%不到点，提升还可以，而且安装和配置也很轻松，还是不错的~

### 参考资料

* [Ninja, a small build system with a focus on speed](https://ninja-build.org/)
