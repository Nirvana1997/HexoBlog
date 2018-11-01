---
title: git原理
category:
  - git
tags:
  - git原理
date: 2018-08-30 17:31:07
---

## 1.前言

一直觉得git很神奇,他的分支切换很快,他的自动合并很智能,他在协作编码时有着不可替代的作用.所以这两天想研究下git的实现原理,参考的资料主要是[Git Internals](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain),《Pro Git》的Chapter 10,还有中文版的[Git内部原理](https://git-scm.com/book/zh/v1/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E5%BA%95%E5%B1%82%E5%91%BD%E4%BB%A4-Plumbing-%E5%92%8C%E9%AB%98%E5%B1%82%E5%91%BD%E4%BB%A4-Porcelain).

<!-- more -->

## 2.git目录组成

在创建或者拉取一个git仓库以后,我们会发现其中会有一个隐藏文件夹.git,我们的版本控制就是靠这个.git文件夹了.

.git目录内有以下文件夹:

![git目录](git原理(一)/git目录.png)

重要的文件或目录主要有这几个:

* HEAD: 指向项目当前分支的文件
* index: 保存暂存区域信息的文件
* objects: 存储所有数据内容的目录
* refs: 存储指向数据(分支)的指针的目录
* hooks: 存储钩子相关数据的目录

## 3.git对象

git底层其实是一套内容寻址文件系统.就是说git底层允许存入任何形式的内容,并返回一个键值,可以随时根据该键值取出存入的内容.

可以通过git的底层命令`hash-object`和`cat-file`存取git对象.

git对象主要有tree对象,文件对象,commit对象.按我的理解,git对象的组织就像是linux的文件系统外再加一个commit对象,指向每次commit对应的"根目录",tree就对应linux中的目录.

书的[第二章](https://git-scm.com/book/zh/v1/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)中做了个实验,以底层的指令完成`git add`和`git commit`的操作,完成后git的对象是这样的:

![git-trial](git原理(一)/git-trial.png)

