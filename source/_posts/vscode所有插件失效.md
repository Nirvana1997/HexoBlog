---
title: vscode所有插件失效
category:
  - vscode
tags:
  - vscode
date: 2021-08-07 23:42:50
---

最近遇到个比较奇怪的问题，vscode所有插件失效了，给解决方案留个档~
<!-- more -->

## 1.问题

这次遇到的问题是vscode的所有插件都失效了，无论旧的重装还是直接装新的，功能都没法使用，用快捷键调用功能时也会报“commond ’xxx‘ not found”的错误。

## 2.解决方法

经过查阅，根据[《VSCode所有插件失效解决log》](https://blog.csdn.net/qq_40657321/article/details/112985537)中的方法，将用户目录下的.vscode文件夹删除后再重启就解决了。

![文件夹](vscode.png)

这个.vscode文件夹主要存的是vscode的插件和一些配置信息，我在坏之前因为误删重装过新版的vscode，猜想可能是因为误删时，.vscode文件夹保留了下来，然后又因为版本冲突导致错误，所以删除让vscode重新生成就解决了。

## 3.总结

软件问题大都可以通过删掉重建，删掉重装解决~😏

