---
title: Godot4配置表生成器
category:
  - Godot
tags:
  - Godot
date: 2022-07-19 18:58:32
---

为了方便开发，我找到了一个大佬的Godot的配置表生成器[Config-Table-Plugin](https://github.com/rayxuln/Config-Table-Plugin)，可以通过json定义结构生成excel配置表，然后按照excel配置表生成Godot中的DataTable。
<!-- more -->

## 1. 使用方式

该配置工具的使用方式作者有视频介绍：[【蘩】[Godot教程] Excel表格转GDScript插件的使用](https://www.bilibili.com/video/BV1fP4y1c7mt)

## 2. Godot4

该工具用到了gdscript，而Godot4中的gdscript有语法有很多更新，因此不兼容，因此我改了下，[
Config-Table-Plugin-Godot4-](https://github.com/Nirvana1997/Config-Table-Plugin-Godot4-)，支持了下Godot4，不过在生成已存在文件，需要删除时会有个报错，需要把原文件先删除，目前还没解决，感觉可能是Godot4的测试版的问题，等正式版出来我再试试~

