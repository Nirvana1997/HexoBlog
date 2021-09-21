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
class A
{
 public:
  template <class T>
  void print(T t);
};
```

TemplateTest.cpp:

```cpp
template <class T>
void print(T t)
{
  cout << typeid(t).name() << endl;
}
```

以上这段代码编译后会发生报错：

```bash
Undefined symbols for architecture x86_64:
  "void A::print<A>(A)", referenced from:
      _main in TemplateTest-f7055f.o
ld: symbol(s) not found for architecture x86_64
```

是编译器在链接时找不到print函数的定义。

## 2.分析原因

让我们回顾一下c++大致的编译过程：

![编译过程](/Users/qianzhihao/HexoBlog/source/_posts/c-模板实现分离/编译过程.png)



### 参考资料

* [C++ 中Template 类、函数的编译过程](https://blog.csdn.net/xianxjm/article/details/73457412)

* [C++ 模板类的声明与实现分离问题（模板实例化）](https://blog.csdn.net/weixin_40539125/article/details/83375452)
