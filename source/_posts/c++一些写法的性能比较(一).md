---
title: c++一些写法的性能比较(一)
category:
  - c++
tags:
  - c++
date: 2022-02-20 15:04:20
---

性能一直是服务器追求的最重要的指标之一，在实际项目中经常会有优化性能的一些工作。其实很多时候一个写法的不同都会多多少少影响性能。因此想在这个系列总结下平时遇到的一些性能相关的写法的比较。
<!-- more -->

## 1.for循环是否判空

在遍历一个可能是空的set时，我一直时习惯不判空的，不过突然有同事说不判空好像会影响性能，于是这次写了3个函数测试了下：

```c++
int global = 0;
void funcX()
{
  global++;
}

void checkEmpty(const set<int>& setNum)
{
  for (int i = 0; i < X; i++)
  {
    if (setNum.empty() == false)
    {
      for (int j : setNum)
      {
        funcX();
      }
    }
  }
}

void noCheckEmpty(const set<int>& setNum)
{
  for (int i = 0; i < X; i++)
  {
    for (int j : setNum)
    {
      funcX();
    }
  }
}

void useIterator(const set<int>& setNum)
{
  for (int i = 0; i < X; i++)
  {
    for (auto it = setNum.begin(); it != setNum.end(); it++)
    {
      funcX();
    }
  }
}
```

以O0的方式每个函数运行10万次，最终结果如下：

```bash
empty:8.546ms
no_empty,foreach:49.532ms
no_empty,iterator:46.798ms
```

可以看到在O0、集合为空时，判一次空还是能提升不少性能的，而foreach循环和取迭代器的循环性能几乎相同，也就是说foreach循环对于空的集合还是会取一次迭代器的。

但是在O2的方式下运行，结果如下：

```c++
empty:0.716ms
no_empty,foreach:1.008ms
no_empty,iterator:1.066ms
```

当集合有1个元素时，结果如下：

```cpp
empty:2.538ms
no_empty,foreach:2.353ms
no_empty,iterator:2.36ms
```

可以看到在O2的编译方式下，是否判空对效率影响不大，在为空时判空更快，在非空时不判空更快，因此是否判空没有太大所谓。不过平时大多数情况下，集合并不为空，所以基本不判空的效率会好一点，写起来额更简洁一点。

## 2.取当前时间

程序中经常需要取当前的时间，一般我们会通过`gettimeofday()`或者是`time()`等函数去获取当前的时间。这些库函数其实本身挺快的，但是在一个工程量比较庞大的项目中，每一帧的运行中可能会非常多次地去取当前时间，或多或少也许会影响到效率。其实对于很多对时间精准度要求没有那么高的地方，可以用一个全局变量去缓存当前时间，每一帧更新一次，这样每一帧去通过库函数取时间的频率就会大大降低，而且功能一般也不会受到影响，除非极端情况每帧的运行时间过长。

例如可以像这样：

```cpp
```

