---
title: folly-FBVector
category:
  - c++
tags:
  - c++
  - folly
  - 学习笔记
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

fbvector最终使用的系数是1.5。并且fbvector也做了优先使用jemalloc来分配内存的处理，因为jemalloc支持就地重新分配（in-place reallocation）。

### ii.对象的可重定位性（Relocation）

FBVector的文档中提到，很多时候大家使用c\+\+时，会保守地把c\+\+的变量假设为不可重定位的。而不可重定位的对象在进行移动时需要进行重新创建拷贝对象、析构原对象的操作，假如vector的元素有这一层限制的话，会一定程度上降低移动元素时的效率。但实际很多情况下c\+\+的对象都是可重定位的，例外只有：

1. 类中使用了内部对象的指针，如：

   ```cpp
       class Ew {
         char buffer[1024];
         char * pointerInsideBuffer;
       public:
         Ew() : pointerInsideBuffer(buffer) {}
         ...
       }

2. 有外部的观察者观察着该对象且持有其指针。（感觉意思应该是构造时注册观察者的情况，假如不是的话，这种持有指针的情况即使重新构造对象也无法避免野指针）

### iii.初始化大小

std::vector默认的初始化容量是1，但是fbvector初始化时会申请至少64字节的空间，从而降低重新申请空间的频率。

## 3.实现细节

### i.支持relocate

为了让重定位更安全，folly使用了一个IsRelocatable的模板：

声明：

```cpp
template <class T>
struct IsRelocatable
    : std::conditional<
          is_detected_v<traits_detail::detect_IsRelocatable, T>,
          traits_detail::has_true_IsRelocatable<T>,
          is_trivially_copyable<T>>::type {}
```

使用：

```cpp
// at global namespace level
namespace folly {
   struct IsRelocatable<Widget> : boost::true_type {};
}
```

### ii.make_window

对于合并数组相关的操作，std::vector的支持比较有限，我印象中只有`vec1.insert(vec2.begin(), vec2.end())`这样整个插入后方的操作。

fbvector提供了`make_window`的方法，给合并数组带来了一定的便利：

```cpp
  //    123456789______
  //        ^
  //        make_window here of size 3
  //
  //    1234___56789___
  void make_window(iterator position, size_type n);
```

这样就可以灵活地在vector中预留空间做一些操作，例如`wrap_frame`：

```cpp
  //        START
  //    fbvector:             inverse window:
  //    123456789______       _____abcde_______
  //                          [idx][ n ]
  //
  //        RESULT
  //    _______________       12345abcde6789___
  void wrap_frame(T* ledge, size_type idx, size_type n)
```

不过这个预留空间的函数我想到的也主要就是合并数组时可以用，并不知道有没有什么其他作用，感觉只是用在这种合并数组元素的情境的话应该也不会特地写这些新函数，留个悬念以后看看能不能解决这个疑惑~

## 4.总结

FBVector本身优化的方式还是比较简单清晰的，核心就是对内存管理策略的优化，相比FBString来说总体优化策略还是很容易理解的。不过内部实现由于很多地方是对内存直接操作，所以函数内的操作还是蛮复杂的，而且用到了一些模板编程和很多标准库里不常用的类如conditional等（至少我不太用），接下来感觉得单独研究下，路漫漫其修远兮~

### 参考资料

* [facebook/folly](https://github.com/facebook/folly)

* [JeMalloc](https://zhuanlan.zhihu.com/p/48957114)



