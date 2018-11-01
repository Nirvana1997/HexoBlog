---
title: epoll的实现机制
category:
  - Linux
tags:
  - Linux
  - epoll
date: 2018-11-01 17:26:44
---

## 1.前言

昨天弄懂了epoll是干啥的,今天来研究下它是怎么实现的以及怎么用的.

<!-- more -->

## 2.select和poll

