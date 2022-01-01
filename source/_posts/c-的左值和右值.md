---
title: c++的左值和右值
category:
  - c++
tags:
  - c++
date: 2021-12-25 15:54:34
---

左值和右值一直是c++中两个很常见但是挺难解释的概念，今天想巩固总结下，希望能更好地掌握他们。
<!-- more -->

## 1.简单理解

最简单地理解方式，就是**赋值操作符左边的是左值，右边的是右值**。例如下面的表达式中：

```cpp
int x = 5;
```

x就是左值，5就是右值。

再用抽象、通用一点的说法的话，**左值代表了一个c++内存中的对象**，即左值是有地址的，可以使用取值符；而右值与之相反，**右值是一个表达式**，它无法对应一个内存中的值。

## 2.官方概念

官方为了定义左值和右值解释了很多，甚至引入了glvalue（“generalized” lvalue）、prvalue（“pure” rvalue）、xvalue（an “eXpiring” value）3个类型来解释，感觉只是几个帮助理解的概念，不需要(¦3」∠)_。

所以这里就整理了下官方给出的例子，用来以后查阅：。

左值：

![lvalue](lvalue.png)

右值（prvalue+xvalue）：

![prvalue](prvalue.png)

![xvalue](xvalue.png)

## 3.总结

左值右值也许我们在编程过程中很少需要关注，但理解一下这几个术语对看文档还是挺有帮助的~

### 参考资料

* [Value categories - cppreference.com](https://en.cppreference.com/w/cpp/language/value_category)
