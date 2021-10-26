---
title: 开源仓库folly研究（一）——总览
category:
  - c++
tags:
  - c++
  - folly
date: 2021-10-03 21:34:40
---

folly是facebook的一个开源仓库，里面有很多兼具效率和可用性的工具类，接下来一段时间想好好研究下这个开源仓库，这次先大概了解下~
<!-- more -->

## 1.folly是什么

folly含义是Facebook Open Source Library，是facebook的工程师们为了减少重复造轮子而开发的一个对stl和boost进行扩充的一个库，在开发时，以可用性和高效性为主要目标，这也正是我们平时写组件时最重要的两个特性，因此这个库
