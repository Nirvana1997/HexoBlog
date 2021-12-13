---
title: 'folly-FBString'
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

### i.eager copy：

eager copy是最简单也是最暴力的字符串数据维护方式。当一个字符串进行拷贝操作时，直接将数据拷贝一份，将数据指针指向拷贝的对象。

这种实现虽然很简单，不用考虑各种复杂的情况，但是当字符串的长度很长时，在空间和时间上都会浪费很多效率。

### ii.COW(copy-on-write)

COW的含义是当对目标字符串需要修改时再进行拷贝，因此拷贝的字符串未进行修改时，和被拷贝字符串使用的是同一片共享的数据。这种实现需要对每个字符串对应的数据计数，当一块数据没有字符串使用以后就将它回收，和c++中的智能指针很类似。

这种方式处理较长的字符串时，可以避免不需要的拷贝，节省效率，但是由于需要维护一个refcount，这个操作是一个原子操作，频繁进行拷贝的话，会比较消耗效率。

关于COW，我看到了个比较有意思的情况：

```cpp
std::string s("str");
const char* p = s.data();
{
    std::string s2(s);
    (void) s[0];
}
std::cout << *p << '\n';
```

下面这段代码在不使用COW时可以正常运行，但是用使用COW的gcc版本去执行会发现p变成了一个野指针。这是因为s2构造时，与s共享了同一片数据，但使用非const的[]操作符取出的对象，编译器会认为你要对其进行修改，所以会重新拷贝一份“str”的数据并让s指向新的数据，然后当代码走出中间一个代码段后，s2的生命周期到了，而它指向的字符串数据没有被共享，因此p所指向的数据被析构了，p也变成了野指针。

不过我用4.8.5版本的gcc试了以后貌似不会野指针，不过s确实重新拷贝了数据：

```cpp
std::string s("str");
const char* p = s.data();
{
  std::string s2 = s;
  cout << (s.c_str() == s2.c_str()) << '\n';
  (void)s[0];
  cout << (s.c_str() == s2.c_str()) << '\n';
}
cout << (p == s.c_str()) << endl;
std::cout << *p << '\n';
```

输出结果：

```bash
> ./test
1
0
0
s
```

不过值得注意的一点是：COW只在拷贝的时候触发：

```cpp
std::string s1 = "4444";
std::string s2 = s1;
cout << ((unsigned long)s1.c_str() == (unsigned long)s2.c_str()) << endl;
```

这样结果输出的是1，但是仅仅数据一样的话：

```cpp
std::string s1 = "4444";
std::string s2 = "4444";
cout << ((unsigned long)s1.c_str() == (unsigned long)s2.c_str()) << endl;
```

输出的就是0了。

在Lua对string的处理中，将string计算hash存入一个hashtable中，所以相同数据的字符串对应数据只会是同一份。

### iii.SSO

由于大多数字符串比较短，所以使用eager copy的成本较低，所以为了效率考虑，出现了SSO（Small String Optimization）这种优化方式。

SSO的优化方式主要是对于较短的字符串的直接存在string类的内部，对于较长的字符串则使用COW的共享数据的方式。

据网上说，gcc5开始禁止了COW，开始使用SSO。

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
5. `find()`使用简化版的**[Boyer-Moore algorithm](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Boyer%E2%80%93Moore_string-search_algorithm)**。在查找成功的情况下，相对于`string::find()`大致有 30 倍的性能提升。在查找失败的情况下也有 1.5 倍的性能提升。
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

FBString使用了一个union去存储短字符串的数据或是中长字符串的指针、长度、容量，这3者在64位系统中正好占用24字节，因此短字符串的长度最长就是23字节。可以看到这个union里数组的大小都是通过`sizeof`表达式去计算出来的，这样就保证了不同环境下的稳定性。

包括一些常数数据，如大中小字符串长度的分割长度节点，FBString中也是使用表达式去计算出来的，计算结果使用`constexpr`常量，即编译期的常量去保存，这样也避免了代码中出现魔法数字的问题：

```cpp
  constexpr static size_t lastChar = sizeof(MediumLarge) - 1;
  constexpr static size_t maxSmallSize = lastChar / sizeof(Char);
  constexpr static size_t maxMediumSize = 254 / sizeof(Char);
  constexpr static uint8_t categoryExtractMask = kIsLittleEndian ? 0xC0 : 0x3;
  constexpr static size_t kCategoryShift = (sizeof(size_t) - 1) * 8;
  constexpr static size_t capacityExtractMask = kIsLittleEndian
      ? ~(size_t(categoryExtractMask) << kCategoryShift)
      : 0x0 /* unused */;
```

然后根据上面计算的节点，FBString在创建时会根据长度进行分类：

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

最终类型会被保存在最后一个字节中：

```cpp
  typedef uint8_t category_type;

  enum class Category : category_type {
    isSmall = 0,
    isMedium = kIsLittleEndian ? 0x80 : 0x2,
    isLarge = kIsLittleEndian ? 0x40 : 0x1,
  };

  Category category() const {
    return static_cast<Category>(bytes_[lastChar] & categoryExtractMask);
  }
```

Category定义`isSmall = 0`其实利用了字符串数据的最后都是0的特性，而对于中大字符串也是在`capacity_`中利用大小去判断，也算是巧妙地节省了保存类型的空间。

### ii.引用计数线程安全

这个线程安全的实现依赖的是`atomic`：

```cpp
  struct RefCounted {
    std::atomic<size_t> refCount_;
    Char data_[1];
  };
```

### iii.jemalloc友好

folly整个库很多地方会使用jemalloc的非标准扩展接口，因此使用时需要特殊处理。

在Malloc.h中有很复杂的对是否使用jemalloc的判断。

``` cpp
/**
 * Determine if we are using jemalloc or not.
 */
#if defined(FOLLY_ASSUME_NO_JEMALLOC) || FOLLY_SANITIZE
  inline bool usingJEMalloc() noexcept {
    return false;
  }
#elif defined(USE_JEMALLOC) && !FOLLY_SANITIZE
  inline bool usingJEMalloc() noexcept {
    return true;
  }
#else
FOLLY_NOINLINE inline bool usingJEMalloc() noexcept {
  ...
}
```

首先会通过FOLLY_ASSUME_NO_JEMALLOC和USE_JEMALLOC两个选项直接指定是否使用jemalloc。假如这两个选项都没有指定的话，则会经过通过一些检测去判断是否能使用jemalloc。

```cpp
FOLLY_NOINLINE inline bool usingJEMalloc() noexcept {
  static const bool result = []() noexcept {
    // Some platforms (*cough* OSX *cough*) require weak symbol checks to be
    // in the form if (mallctl != nullptr). Not if (mallctl) or if (!mallctl)
    // (!!). http://goo.gl/xpmctm
    if (mallocx == nullptr || rallocx == nullptr || xallocx == nullptr ||
        sallocx == nullptr || dallocx == nullptr || sdallocx == nullptr ||
        nallocx == nullptr || mallctl == nullptr ||
        mallctlnametomib == nullptr || mallctlbymib == nullptr) {
      return false;
    }
    ...
  }();
  return result;
}
```

if条件语句中的变量都是函数指针，可以看到这个函数主要是判断一系列函数指针是否有值。剩下的看起来应该是一些特殊情况的处理。

而这些函数指针在检测到引用了jemalloc的情况下是非空的。folly使用CMake的CHECK_INCLUDE_FILE_CXX去判断是否使用了jemalloc：

```cmake
if (CMAKE_SYSTEM_NAME STREQUAL "FreeBSD")
  CHECK_INCLUDE_FILE_CXX(malloc_np.h FOLLY_USE_JEMALLOC)
else()
  CHECK_INCLUDE_FILE_CXX(jemalloc/jemalloc.h FOLLY_USE_JEMALLOC)
endif(
```

### iv.find函数的优化

FBString对`find`函数的优化主要是使用了Boyer-Moore算法的思路（[字符串匹配的Boyer-Moore算法](https://www.ruanyifeng.com/blog/2013/05/boyer-moore_string_search_algorithm.html)）。不过原版比较复杂，适合去搜索长字符串，常用于网页、文本的搜索，对于目标字符串较短的情况反而有点得不偿失，因此FBString的`find`是它的简化版：

```cpp
template <typename E, class T, class A, class S>
inline typename basic_fbstring<E, T, A, S>::size_type
basic_fbstring<E, T, A, S>::find(
    const value_type* needle,
    const size_type pos,
    const size_type nsize) const {
    ...
    // 前面逐位对比，直至i位置开始的字符串最后一位和needle最后一位相同
    for (size_t j = 0;;) {
      assert(j < nsize);
      if (i[j] != needle[j]) {
        // 然后这里会计算需要跳过的位数，不过没有Boyer-Moore那样对好后缀进行判断，仅是对最后一位字符找needle中相同的一位对齐
        if (skip == 0) {
          skip = 1;
          while (skip <= nsize_1 && needle[nsize_1 - skip] != lastNeedle) {
            ++skip;
          }
        }
        i += skip;
        break;
      }
      if (++j == nsize) {
        // 找到目标
        return i - haystack;
      }
    }
}
```

### v.与std::string的兼容

c++中的`std::string`其实是` basic_string<char>`的别名，FBString为了兼容它，实现了一系列重载函数：

```cpp
  // std::string转FBString
  template <typename A2>
  basic_fbstring& operator=(const std::basic_string<E, T, A2>& rhs) {
    return assign(rhs.data(), rhs.size());
  }

  // FBString转std::string
  std::basic_string<E, T, A> toStdString() const {
    return std::basic_string<E, T, A>(data(), size());
  }

  // 比较函数 
  template <typename E, class T, class A, class S, class A2>
  inline bool operator==(
      const basic_fbstring<E, T, A, S>& lhs,
      const std::basic_string<E, T, A2>& rhs) {
    return lhs.compare(0, lhs.size(), rhs.data(), rhs.size()) == 0;
  }

  ......
```

而且`std::string`的常用函数在FBString基本是以一样的形式去定义的，如`size()`、`capacity()`、`empty()`、`begin()`、`end()`等。而且看注释和记录，FBString在string的函数更新后也会相应地更新去兼容。

这些函数保证了和原有的标准库的std::string的兼容性，在使用时完全可以当成std::string去用，甚至很多时候直接可以把旧代码的一个std::string替换成fbstring，它们之间的比较和相互转换都是一个表达式就能完成的。当要研发一个旧系统的新版本时，兼容性是一个很重要但也很难完美满足的一个方面，FBString的兼容性真的做的很好（不过工作量也确实挺大的）。

## 4.总结

FBString这么整体看下来，可以看得出来还是有很多细节的。感觉看下来这个库的代码风格整体很严谨，大致有这些特点：

1. 几乎所有复杂接口都加上了assert验证正确性，保证可靠性。
2. 很多地方也对大小端进行了区分，常量尽可能用系统变量计算，长度均使用size_type，尽可能的保证了库的可移植性。
3. 代码对内存的使用十分严格，能用一个uint8解决的问题绝对不会不用uint32。
4. 使用了非常多模板定义的类和函数，提高可复用性。（不过感觉有点过多了，不知道是不是真的都需要抽出来，可能有点过度设计，用的多的话也可能导致编译过慢，个人理解）

folly确实有很多可以借鉴的地方，接下来继续研究ヾ(◍°∇°◍)ﾉﾞ！

### 参考资料

* [facebook/folly](https://github.com/facebook/folly)
* [C++ folly库解读](https://zhuanlan.zhihu.com/p/348614098)
* [GNU C++ 标准库中 string 实现简介](https://zhuanlan.zhihu.com/p/344859791)
* [Legality of COW std::string implementation in C++11](https://stackoverflow.com/questions/12199710/legality-of-cow-stdstring-implementation-in-c11)
* [c++再探string之eager-copy、COW和SSO方案](https://www.cnblogs.com/cthon/p/9181979.html)
* [一篇文章搞懂STL中的空间配置器allocator](https://www.coonote.com/cplusplus-note/space-allocator.html)
* [分配器——allocators](https://www.cnblogs.com/area-h-p/p/12020879.html)

