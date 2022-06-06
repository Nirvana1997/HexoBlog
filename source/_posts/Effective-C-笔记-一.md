---
title: Effective C++笔记(一)
category:
  - c++
tags:
  - c++
  - 学习笔记
date: 2022-05-29 15:21:10
---

前一段时间一直疫情，在家做饭洗碗好费时间，工作也好忙，学的东西就比较碎，好久没写博客了hh。

正好前段时间在看《Effective C++》，在工作一段时间后再看，感觉之前还是有很多不太注意的细节，这次的笔记想把其中提到的所有条款概括记录一下加强下记忆。

<!-- more -->

## 条款1：视C++为一个语言联邦

c\+\+继承了结构化语言c的特性，自身扩展出了C with Class的面向对象特性，还有Template C\+\+的泛型编程方式和对STL程序库的使用，这几种成分都融合在c\+\+中，因此对于不同成分需要使用不同的风格（不过我其实感觉平时使用stl时没有什么特别的规范，可能还没有意识到他的特殊，等后面了解了再来总结）

## 条款2：尽量以const，enum，inline，替换#define

书中提到这点主要是由于以下原因：

> \# define ASPECT_RATIO 1.653
>
> const double AspectRatio = 1.653;
>
> 此外对浮点常量而言，使用常量可能比是要用#define导致较小量的码，因为预处理器“盲目地将宏名称ASPECT_RATIO替换为1.653”可能导致目标码出现多份1.653，若改用常量AspectRatio绝不会出现相同的情况。

但是经过测试：

```cpp
const double I_NUM = 1.653;
// #define I_NUM 1.653;

double func(double i)
{
  return i;
}

int main()
{
  double i1 = I_NUM;
  double i2 = I_NUM;
  double i3 = I_NUM;

  func(i1);
  func(i2);

  return 0;
}
```

这段代码不管使用const还是#define，在c\+\+11和c\+\+98标准下，编译出的汇编是完全一致的：

![constDefineCompare](/Users/qianzhihao/HexoBlog/source/_posts/Effective-C-笔记-一/constDefineCompare.png)

编译出来的可执行文件大小也完全一致：

![constTest](/Users/qianzhihao/HexoBlog/source/_posts/Effective-C-笔记-一/constTest.png)

所以我猜这个弊端可能已经被编译器优化了。

然后第二个#define的弊端是#define假如不用#undef的话默认作用域是整个文件，不够灵活，不像const静态变量可以通过放入类通过private、public等权限修饰词或是写成全局变量去控制其作用范围。

但是const静态变量有一个问题是声明数组时，大小需要是一个编译期常量，此时假如不使用宏的话，书中提到了一个可以解决的技巧：the enum hack。

```cpp
class GamePlayer {
  private:
    enum { NumTurns = 5 };
    int scores[NumTurns];
};
```

通过使用enum，可以创建出一个编译期间的常量，解决const静态变量无法作为数组大小的问题。

## 3.尽可能使用const

这点挺好理解也挺好实践，和权限最小化原则是一个道理，每个函数、每个成员的权限尽可能低可以一定程度减少错误的发生。

## 4.确定对象被使用前已先被初始化

这点也是学C\+\+的老生常谈了~（不过我至今不太理解C\+\+为什么不给类中的基础变量提供初始值。假如为了效率不提供初始值可以不初始化的话，其实牺牲了程序的稳定性，所以实际现在使用都会加上初始值，其实也没省到效率，有点费解。）

## 5.总结

以上就是《Effective C\+\+》第一章的内容了，虽然书很老了但不得不说还是能有所收获的，有些规则可能如今比较普遍或者过时了，但还是值得思考和研究的。

### 参考资料

* 《Effective C\+\+》—— Scott Meyers
