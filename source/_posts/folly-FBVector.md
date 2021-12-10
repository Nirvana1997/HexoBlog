---
title: folly-FBVector
category:
  - c++
tags:
  - c++
  - folly
date: 2021-12-04 23:53:57
math: true
---

上次研究了folly的FBString，还是有很多体会的，这次来研究下folly的FBVector。

<!-- more -->

## 1.简介

std::vector的实现很简单，就是个可以动态扩展的数组，而FBVector在介绍中，号称当你将项目中的std::vector替换成folly:fbstring在所有情况下都不会有性能上的副作用，绝大多数情况都是能提升性能的，而且某些时候提升的会很可观。

## 2.特性

### i.内存处理策略

std::vector在push_back的时候如果容量不足，会重新申请内存，而重新申请内存时扩大的系数是2。我们可以计算下所有这会导致每次申请的新内存永远比之前每次扩大前的原内存大小之和更大，因为：

$$
1 + 2^1 + 2^2 + 2^3... + 2^n = 2^{n+1} - 1 < 2^{n+1}
$$
因此导致新申请的内存没法重新利用之前申请过的内存块，只能新开辟一块。假设扩大倍数为r，通过计算可以得到：
$$
之前n次申请内存大小总和：\sum_{i=0}^{n}r^i=\frac{r^{n+1}-1}{r-1}\\
n+1次申请大小：r^{n+1}\\
相差：\Delta d=\frac{(2-r)r^{n+1}-1}{r-1}\\
由于每次是扩大，所以r>1,要使\Delta d>0，只需要(r-1)\Delta d=(2-r)r^{n+1}-1>0\\
即2-r>\frac{1}{r^{n+1}}\\
即r+\frac{1}{r^{n+1}}<2
$$
通过上面的不等式，可以得出结论，1<r<2时，n只要足够大$\Delta d$是可以为正的，也就是n+1次申请的内存是用上之前的申请过的空间的。当然真的n需要很大才能满足的话，n+1次之前的内存应该也已经分配出去了，但大多数情况下要满足这个条件n不用太大就能满足。（第一次知道markdown可以写这种数学符号和公式，打开新世界的大门🥳）

fbvector最终使用的系数是1.5。并且fbvector也做了优先使用jemalloc来分配内存的处理，因为jemalloc支持就地重新分配（in-place reallocation）

### 参考资料

* [facebook/folly](https://github.com/facebook/folly)

* [JeMalloc](https://zhuanlan.zhihu.com/p/48957114)



