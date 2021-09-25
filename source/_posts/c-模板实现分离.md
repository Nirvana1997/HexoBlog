---
title: c++模板实现分离
category:
  - c++
tags:
  - c++
  - template
date: 2021-09-14 16:41:01
---

最近使用c++的模板实现一个成员函数时，将定义和实现分别放在了.h和.cpp文件，结果在编译最后的链接过程时，发生了报错，这次就针对这个问题研究下。
<!-- more -->

## 1.报错信息

TemplateTest.h:

```cpp
template <class T>
void print(T t);
```

TemplateTest.cpp:

```cpp
#include "TemplateTest.h"

#include <iostream>

template <class T>
void print(T t)
{
  std::cout << typeid(t).name() << std::endl;
}

```

main.cpp:

```bash
#include "TemplateTest.h"

int main()
{
  int i = 0;
  print(i);
  return 0;
}
```

以上这段代码编译后会发生报错：

```bash
Undefined symbols for architecture x86_64:
  "void print<int>(int)", referenced from:
      _main in main-96b07f.o
ld: symbol(s) not found for architecture x86_64
```

是编译器在链接时找不到print函数的定义。

## 2.分析原因

让我们回顾一下c++大致的编译过程：

![编译过程](编译过程.png)

在编译时，对于普通的函数，在当前文件找不到函数定义，则在函数位置生成符号，在链接时，再寻找这个函数的定义。
而模板是两次编译。第一次编译时只对模板进行编译，不生成具体函数，在调用时才直接生成具体函数，因此需要在调用时获取到实现，找不到就会报上面的错。

## 3.解决方案

解决这个报错的方式主要有几种：

1. 在声明处直接实现模板函数。
2. 在使用模板函数的cpp文件中实现模板函数，不过多个文件中使用得分别实现，比较麻烦。
3. 新建一个文件，网上一般会以.hpp或者.tpp作为后缀，实现模板函数，在使用处include。

## 4.总结

c++的模板实现与java的泛型还有有不少差别的，有很多细节，而且由于每次使用时都会生成实例化代码，所以滥用容易导致编译后体积变大，编译速度变慢等问题，使用时还是需要多研究研究。

### 参考资料

* [C++ 中Template 类、函数的编译过程](https://blog.csdn.net/xianxjm/article/details/73457412)

* [C++ 模板类的声明与实现分离问题（模板实例化）](https://blog.csdn.net/weixin_40539125/article/details/83375452)

* [C++服务编译耗时优化原理及实践](https://tech.meituan.com/2020/12/10/apache-kylin-practice-in-meituan.html)

