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

读了很多讲解epoll的文章,几乎都会提到select和poll,因为他们做的事是相同的,就是实现IO多路复用.

IO多路复用简单来说就是使用一个进程同时处理多个流,在服务器上"流"通常是socket.所以IO多路复用的核心问题便是怎么处理这多个流,要知道是否有需要处理的流,现在需要处理哪个流.

select和poll的实现便是遍历去判断每个流是否需要处理.当遇到某个流需要处理时,便唤醒处理进程.

select的函数定义是这样子的:

```c++
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

fd_set是作为参数传入的,因为是系统调用,所以在调用select和poll时,会将所有监控的流的fds从用户内存拷贝到内核内存空间.

由于这样的实现,使得select和poll会产生以下问题:

因为需要拷贝操作和遍历,随着fds中的文件描述符数目增长,耗费的性能会线性增长,而且select监控的文件描述符有1024的限制.poll解决了1024这一限制,但性能的问题依旧没能解决,所以最终都被epoll所代替了.

## 3.epoll

重点来了,因为poll没能解决select的性能问题,所以出现了epoll.

上面提到select和poll的性能问题,主要是因为两个原因:fds的拷贝和对fds的遍历,下面就来看看epoll如何解决这两个问题.

首先先看下epoll的接口:

```c++
/** 创建epoll,返回epfd(epoll会占用一个fd) */
int epoll_create(int size);
/** 对epoll监控的fds进行操作,op表示操作,有增、删、改的操作,event表示需要监听什么事件,下面具体展开 */
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
/** 返回就绪的事件数 */
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

可以看到epoll不需要每次将整个fd_set传入,只需要将修改的告诉epfd对应的epoll就行了.而且epoll有一个mmap的内存映射机制,使得用户空间的一块地址和内核空间的一块地址同时映射到相同的一块物理内存地址,这样也避免了内存的拷贝.并且epoll将监控文件fd用红黑树的形式保存下来,进一步提高了修改监控文件集合的效率.(2.6.8之前的内核使用hashtable,所以需要指定size,2.6.8以后size实际上没有什么意义了)

epoll为了避免遍历所有监控的fd,引入了一个ready_list,保存就绪的事件,同时每个epoll会有一个单独的睡眠列表(select和poll在事件发生时会遍历总的睡眠列表调用回调函数).这样大大减少了遍历操作的成本.

## 4.总结

看完实现,我的理解大概是这样的:

把内核比作一个中转站的话,用户程序就是提货人.内核接受货物并交给用户程序进行对应操作,可能是多个.

每个提货人手里会有一份需要接受的货物清单.

select和poll就是提货人把清单复印给中转站,然后中转站去挨个检查清单上的货物,若有接受到的,就检查所有等待的用户,哪个用户需要这个货物,去通知他们.

而epoll则为每个用户添加了一个管理员,他可以看到用户的清单,并且他能知道货物到来可能需要的用户,效率会高很多.

(上面讲的实现机制比较简单,主要参考[大话 Select、Poll、Epoll](https://cloud.tencent.com/developer/article/1005481),里面十分详细~)



