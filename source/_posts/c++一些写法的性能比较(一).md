---
title: c++一些写法的性能比较(一)
category:
  - category
tags:
  - tag
date: 2022-02-20 15:04:20
---

性能一直是服务器追求的最重要的指标之一，在实际项目中经常会有优化性能的一些工作。其实很多时候一个写法的不同都会多多少少影响性能。因此想在这个系列总结下平时遇到的一些性能相关的写法的比较。
<!-- more -->

### 1.for循环是否判空

在遍历一个可能是空的set时，我一直时习惯不判空的，不过突然有同事提出是否判空会影响性能，于是这次测试了下：

```c++
  set<int> setNum;
  // empty
  for (int i = 0; i < X; i++)
  {
    if (setNum.empty() == false)
    {
      for (int j : setNum)
      {
      }
    }
  }

  // no_empty,foreach
  for (int i = 0; i < X; i++)
  {
    for (int j : setNum)
    {
      test++;
    }
  }

  // no_empty,iterator
  for (int i = 0; i < X; i++)
  {
    for (auto it = setNum.begin(); it != setNum.end(); it++)
    {
      test++;
    }
  }
```

以O0的方式运行，最终结果如下：

```bash
empty:8.546ms
no_empty,foreach:49.532ms
no_empty,iterator:46.798ms
```

可以看到在O0、集合为空时，判一次空还是能提升不少性能的，而foreach循环和取迭代器的循环性能几乎相同，也就是说foreach循环对于空的集合还是会取一次迭代器的。

但是在O2的方式下运行，结果如下：

```c++
empty:1.474ms
no_empty,foreach:1.063ms
no_empty,iterator:1.455ms
```

