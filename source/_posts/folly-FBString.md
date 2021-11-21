---
title: 'folly:FBString'
category:
  - c++
tags:
  - c++
  - folly
date: 2021-10-28 23:19:55
---

今天研究folly的字符串的实现——FBString。
<!-- more -->

## 1.字符串实现方式

要实现字符串，关键的是两个数据：

1. 对应字符串数据的地址
2. 字符串的长度

对于这些数据的维护方式，主要的有以下几种方式：

###i.eager copy：

eager copy是最简单也是最暴力的字符串数据维护方式。当一个字符串进行拷贝操作时，直接将数据拷贝一份，将数据指针指向拷贝的对象。

这种实现虽然很简单，不用考虑各种复杂的情况，但是当字符串的长度很长时，在空间和时间上都会浪费很多效率。

### ii.COW(copy-on-write)

COW的含义是当对目标字符串需要修改时再进行拷贝，因此拷贝的字符串未进行修改时，和被拷贝字符串使用的是同一片共享的数据。这种实现需要对每个字符串对应的数据计数，当一块数据没有字符串使用以后就将它回收，和c++中的智能指针很类似。

这种方式处理较长的字符串时，可以避免不需要的拷贝，节省效率，但是由于需要维护一个refcount，这个操作是一个原子操作，频繁进行拷贝的话，会比较消耗效率。

关于COW，我看到了个比较有意思的情况：

```c++
std::string s("str");
const char* p = s.data();
{
    std::string s2(s);
    (void) s[0];
}
std::cout << *p << '\n';
```

下面这段代码在c++11（c++11中禁止了COW）标准下可以正常运行，但是用使用COW的gcc版本去执行会发现p变成了一个野指针。这是因为s2构造时，与s共享了同一片数据，但使用非const的[]操作符取出的对象，编译器会认为你要对其进行修改，所以会重新拷贝一份“str”的数据并让s指向新的数据，然后当代码走出中间一个代码段后，s2的生命周期到了，而它指向的字符串数据没有被共享，因此p所指向的数据被析构了，p也变成了野指针。

### iii.SSO

由于大多数字符串比较短，所以使用eager copy的成本较低，所以为了效率考虑，出现了SSO（Small String Optimization）这种优化方式。c++11开始禁止了COW，开始使用SSO。

SSO的优化方式主要是对于较短的字符串的直接存在string类的内部，对于较长的字符串则使用COW的共享数据的方式。

```c++
  std::string s("01234567890123456789012");
  std::string s2("0123456789012345678901");
  cout << "s: size:" << s.size() << " ";
  cout << "distance:" << ((long)s.data() - (long)(&s)) << endl;
  cout << "s2: size:" << s2.size() << " ";
  cout << "distance:" << ((long)s2.data() - (long)(&s2)) << endl;
```

```bash
s: size:23 distance:-140728624740248
s2: size:22 distance: 1
```

通过以上代码可以发现，字符串长度超过阈值22以后，数据就不在string类内部了，它会存在堆上分配的空间。（字符串长度22加上'\0'符号一共23个字节，所以代表string内部可以容纳23个字节，测试使用的是gcc4.2）

## 2.FBString

FBString就功能上来说和string是一致的，但是在效率方面有一定的优化。std的string对于不同长度的string使用了两层分级策略，而FBString使用了三层分级策略：

1. length≤23字节：使用SSO中对短字符串的处理方式，直接存在string类内部；
2. 23字节＜length≤255字节：使用eager copy的方式，直接拷贝string的数据；
3. length>255字节：使用COW的方式，进行懒拷贝，当数据发生变化的时候再进行实际的数据拷贝。

根据官方文档，FBString有以下特性：

1. 与 std::string 100%兼容。
2. 在对大字符串使用COW方式存储时，对于引用计数线程安全。
3. 使用malloc代替allocators。
4. 对 Jemalloc 友好。如果检测到使用 jemalloc，那么将使用 jemalloc 的一些非标准扩展接口来提高性能。
5. `find()`使用简化版的**[Boyer-Moore algorithm](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Boyer%E2%80%93Moore_string-search_algorithm)**。在查找成功的情况下，相对于`string::find()`有 30 倍的性能提升。在查找失败的情况下也有 1.5 倍的性能提升。
6. 可以与 std::string 互相转换。

## 3.实现细节

### i.字符串数据

FBString的核心成员变量大致有这几个：

```cpp
struct MediumLarge {
    Char* data_;
    size_t size_;
    size_t capacity_;
  };

union {
  uint8_t bytes_[sizeof(MediumLarge)]; // 为了能获取最后一个字节的数据
  Char small_[sizeof(MediumLarge) / sizeof(Char)];
  MediumLarge ml_;
};
```

FBString使用了一个union去存储短字符串的数据或是中长字符串的指针、长度、容量，这3者在64位系统中正好占用24字节，因此短字符串的长度最长就是23字节。

为了给字符串分类，FBString定义了

fbstring在创建时会根据长度进行分类：

```cpp
  fbstring_core(
      const Char* const data,
      const size_t size,
      bool disableSSO = FBSTRING_DISABLE_SSO) {
    if (!disableSSO && size <= maxSmallSize) {
      initSmall(data, size);
    } else if (size <= maxMediumSize) {
      initMedium(data, size);
    } else {
      initLarge(data, size);
    }
    assert(this->size() == size);
    assert(size == 0 || memcmp(this->data(), data, size * sizeof(Char)) == 0);
  }
```

然后

## 4.总结

FBString这么整体看下来，可以看得出来还是有很多细节的。感觉看下来这个库的代码风格很严谨，几乎所有复杂接口都加上了assert验证正确性，很多地方也对大小端进行了区分，做到了尽可能高的可移植性。而且代码对内存的使用十分严格，能用一个uint8解决的问题绝对不会不用uint32。

### 参考资料

* [C++ folly库解读](https://zhuanlan.zhihu.com/p/348614098)

* [GNU C++ 标准库中 string 实现简介](https://zhuanlan.zhihu.com/p/344859791)

* [Legality of COW std::string implementation in C++11](https://stackoverflow.com/questions/12199710/legality-of-cow-stdstring-implementation-in-c11)

* [c++再探string之eager-copy、COW和SSO方案](https://www.cnblogs.com/cthon/p/9181979.html)

* [一篇文章搞懂STL中的空间配置器allocator](https://www.coonote.com/cplusplus-note/space-allocator.html)

* [分配器——allocators](https://www.cnblogs.com/area-h-p/p/12020879.html)

