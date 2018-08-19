---
title: protobuf学习记录(一)——介绍与安装
date: 2018-06-17
category: protobuf
tags: 
- protobuf 
- C++ 
- 序列化
---

## 前言
由mentor介绍，开始接触protobuf，感觉还是蛮有用的一个工具，记录一下学习过程~

<!-- more -->

## 1.protobuf简介
Protocol Buffer，即protobuf，是google推出的实现结构化数据的序列化存储的开源工具。它轻量灵活，且十分高效。

其他序列化手段的有很多不便：

> 1. 以二进制形式存储。这种方式有很大局限性，因为以这种方式存储的数据必须以固定的形式存储，在存储和读取的时候，必须要有同样的存储布局和字节顺序（大小端）等，要求十分苛刻，难以扩展。
2. 将数据项编写为单个字符串。例如将4个整数编写为“12：3：-23：67”。 这是一种简单而灵活的方法，但它需要编写一次性的解析代码，而且解析会需要一定的运行时间成本。 这最适合非常简单的数据。
3. 以XML文件存储。XML文件是一种我们可以直接查看的数据存储形式，而且现在已经有很多很完善的库可以使用。使用XML文件可以很方便地在不同的项目中共享数据。但XML的文本表现手法和标记的符号化会导致文件数据量的增大，同时解析和存储XML文件会给应用性能带来一定影响。而且使用XML这样一种树形结构去存储结构化数据对象这种属性简单的对象在处理时反而变得复杂。

以上内容来源于[官方文档](https://developers.google.com/protocol-buffers/docs/cpptutorial)

## 2.protobuf安装
protobuf可以使用于不同的语言。我以下主要以Mac系统，c++版本的protobuf为例。

### 1.安装必要的UNIX工具
安装过程中需要用到一些UNIX的工具，所以需要先安装xcode的命令行工具。

安装过xcode的ide后，在命令行输入以下命令即可安装：

```
$ sudo xcode-select --install
```
在安装UNIX工具前，还需要安装macport，可以在[https://www.macports.org](https://www.macports.org)下载到该工具。
默认会将macport安装到/opt/local/bin/port。

然后使用以下命令即可完成必要UNIX工具的安装：
```
$ sudo /opt/local/bin/port install autoconf automake libtool
```

### 2.安装protobuf
在安装完必要的UNIX tools以后就可以开始正式安装protobuf了。

安装所需文件的获取可以通过两种方式:

(i).可以直接下载zip包(推荐):
[Release Protocol Buffers](https://github.com/google/protobuf/releases/latest)

(ii).也可以使用以下指令生成安装所需文件：
```
$ git clone https://github.com/google/protobuf.git
$ cd protobuf
$ git submodule update --init --recursive
$ ./autogen.sh
```

我在实际使用第二种方式时，autogen.sh运行时会出现`./autogen.sh: line 32: autoreconf: command not found`的错误，导致无法生成configure文件，使用第一种方式configure文件则直接包含在里面了。

最后使用以下指令完成安装：
```
$ ./configure
$ make
$ make check
$ sudo make install
$ sudo ldconfig # 刷新共享库缓存(可选)
```
使用--version检查是否安装成功：
```
$ protoc --version
libprotoc 3.5.1
```

主要参考[Protocol Buffers C++ Installation](https://github.com/google/protobuf/blob/master/src/README.md)

---
以上就是我刚开始学习protobuf的过程。刚接触protobuf，可能会有些说的不够正确，大家可以联系指正~
