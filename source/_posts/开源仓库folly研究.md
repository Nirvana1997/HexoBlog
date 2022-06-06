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

folly含义是Facebook Open Source Library，是facebook的工程师们为了减少重复造轮子而开发的一个对stl和boost进行扩充的一个库，在开发时，以可用性和高效性为主要目标，这也正是我们平时写组件时最重要的两个特性，因此这个库对于从事c++开发的我们来说还是很有研究价值的~

## 2.folly内容

folly的内容主要包含了优化的stl容器、一些多线程相关的组件，以及一些独立的组件。网上有一份各组件的简介：

> ### i.独立有用的小技巧
>
> Eventfd.h ---- 针对eventfd系统调用的包装器。
>
> Foreach.h ---- 伪语句（作为宏语句来实现），用于迭代。
>
> IntrusiveList.h --- 方便类型定义，用于使用boost::intrusive_list(不知道干什么的)。
>
> Likely.h ---- 针对__builtin_expect的包装器。分支预测编译加速。
>
> Malloc.h ---- 内存分配助手，尤其是使用jemalloc时。
>
> MapUtil.h ---- 用于查找联合容器的小工具，找不到返回默认值。（比如std::map和std::unordered_map）。
>
> Preprocessor.h ---- 获取可变参数的第1个或第2个参数，用于模板编程！Synchronized.h的实现就靠这个！
>
> ScopeGuard.h ---- Basically, it guarantees that a function is executed upon leaving the currrent scope unless otherwise told. 即确保资源能够被正确析构(调用资源析构函数)。
>
> StlAllocator.h ---- STL分配器，包装简单的分配/取消分配接口。貌似为了低版本gcc。
>
> Traits.h ---- 类型特性。用于判断类型是否可直接内存拷贝(可重定位的对象)。C\+\+假定所有的对象都是“non-relocatable values”(需要调用构造函数而不能直接拷贝内存数据)。实际中，很多C\+\+对象可通过直接拷贝内存数据完成对象"再造"！(Relocatable object/type -- 可重定位的对象/类型)。Traits.h的核心就是提供"可重定位的类型"编译时判断工具。FBvector的核心优化之一：利用memcpy/memmove来处理"可重定位的类型"！
>
> ### ii.C++功能增强和扩展
>
> FBString.h ---- std::string性能优化版本。
>
> FBvector.h ---- std::vector性能优化版本。
>
> Bits.h ---- 各种位处理实用组件，针对速度而优化。
>
> Conv.h ---- 各种数据转换例程（尤其是to和from字符串），针对速度和安全进行了优化。
>
> DiscriminatedPtr.h ---- 类似boost::variant，但完全局限于指针。使用指针中最高位、未使用的16位作为鉴别器。所以sizeof(DiscriminatedPtr<int, string, Widget>) == sizeof(void*)。
>
> Dynamic.h ---- 动态类型对象，类似boost::variant。用于json.h。
>
> Format.h ---- Python式样的格式化实用组件。C++功能增强和扩展的集大成者，基本上用到了上述的各个头文件！
>
> Range.h ---- 类Boost的随机访问数据包装类，针对StringPiece的定制版本。
>
> String.h ---- 非常有用的string工具集合：std::string <=> FBstring 互转工具、C风格转义字符串工具(反转工具)、stringPrintf工具、prettyPrint(支持时间、容量等常见单位）、hexDump工具、errnoStr\exceptionStr、demangle(串化C++类型）、split(分拆字符串)。
>
> Unicode.h ---- 定义了codePointToUtf8函数。实现unicode码点到utf-8编码的转换。
>
> ### iii.简化多线程编程
>
> Arena.h，ThreadCachedArena.h ---- 内存分配的简单地方：多次内存分配同时被释放。使用线程版本。简化内存管理，相当于java的gc(垃圾回收机制)。
>
> AtomicHashMap.h，AtomicHashArray.h ---- 高性能的原子哈希图，采用几乎无锁的操作。
>
> ProducerConsumerQueue.h ---- 单生产者单消费者队列。
>
> SmallLocks.h ---- 非常小的旋转锁（1字节和1位）。
>
> Synchronized.h ---- 提供一种非常好的多线程同步编码范式！！！请直接看doc和测试代码！
>
> ThreadLocal.h ---- 改进的线程本地存储，用于存储非内置类型。取代pthread_key_t。
>
> ThreadCachedInt.h ---- 使用线程缓存的高性能原子增量。
>
> ### iv.独立组件
>
> Hash.h ---- 各种流行的哈希函数实现。
>
> GroupVarint.h ---- 针对32位值的Group Varint编码。
>
> Histogram.h ---- 用于收集直方图数据。
>
> Json.h ---- JSON序列化器和反序列化器。使用dynamic.h。
>
> Random.h ---- 只定义了一个函数：randomNumberSeed()。使用当前时间和PID来产生随机数种子。
>
> TimeoutQueue.h ---- 定时器队列。按项目设定超时的队列。
>
> ### v.就是为了性能
>
> PackedSyncPtr.h ---- 一种高度专业化的数据结构，含有指针、1位旋转锁和15位整数，它们都在一个64位整型数中。目标：节约空间(当前64位机的指针高16位未用)。用到SmallLocks。
>
> RWSpinLock.h ---- 快速而紧凑的读取器/写入器旋转锁。
>
> small_vector.h ---- 含有小缓冲器方面的优化vector，策略可选：NoHeap、OneBitMutex。
>
> sorted_vector_types.h ---- 类似std::map的集合体，但是作为排序向量来实现。适用：数量少。目的：节约空间。

## 3.总结

folly可以说是开源的”轮子类“代码中最好的之一了，非常值得研究，接下来一段我应该会多研究研究这个仓库。（不过可能得过一段时间，接下来会看一阵TI10，(*^▽^*)）

### 参考资料

* [folly库的学习心得](https://www.cnblogs.com/lenmom/p/9283031.html)
