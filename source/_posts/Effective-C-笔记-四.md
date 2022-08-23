---
title: Effective C++笔记(四)
category:
  - c++
tags:
  - c++
  - 学习笔记
date: 2022-07-19 12:41:41
---

第四章~
<!-- more -->

## 条款18：让接口容易被正确使用，不易被误用

这点无论在什么语言中都是很重要的规则。

总之要尽可能避免可能的错误，阻止误用，可以通过建立新类型、限制类型，限制传入值等方法。

然后可以通过`shared_ptr`去管理新建的对象，从而防止内存泄漏的风险，同时可以防范DLL问题，可以被用来自动解除互斥锁等。

## 条款19：设计class犹如type

这一条款其实非常宏观，核心思想就是对于设计class要十分谨慎，思考的角度要尽可能全面。作者给出的方式就是思考一系列问题（比较难概括，最好看原文）：

> * 新type的对象应该如何被创建和销毁？
> * 对象的初始化和对象的赋值该有什么样的差别？
> * 新type的对象如果被passed by value，意味着什么？
> * 什么是新type的“合法值”？
> * 你的新type需要配合某个继承图系吗？
> * 你的新type需要什么样的转换？
> * 什么样的操作符和函数对此新type而言是合理的？
> * 什么样的标准函数应被驳回？
> * 谁改取用新type的成员？
> * 什么是新type的“未声明接口”？
> * 你的新type有多么一般化？
> * 你真的需要一个新type吗？

## 条款20：宁以pass-by-reference-to-const替换pass_by_value

## 条款21：必须返回对象时，别妄想返回其reference

这两点其实从现在的眼光来看有些平常了，学c++的引用的时候大概率都会了解到。总的来说，传值时对非基础类型来说尽量传递const引用，因为直接传值会默认使用copy传值的方式传递，会有额外开销。然后不要返回指向local stack对象的指针或引用，这点也是老生常谈了。

## 条款22：将成员变量声明为private

这点就是要将变量都尽可能封装起来，读写等通过函数接口的方式开放，从而方便开放特定权限和减少变更的成本，为class作者提供充分的实现弹性。这里有点要注意下，就是protected这个关键词的封装性其实并不比public好，因为取消一个public的接口会破坏所有调用接口的类，而取消一个protect接口则会破坏所有派生类，这两种的破坏性很多时候都是很致命的。

## 条款23： 宁以non-member、non-friend替换member函数

这点乍一看比较反直觉，但其实本质还是尽可能提高封装性。这里需要注意，friend non-member的函数和member函数封装性是等同的，都可以直接访问private变量，一定是non-member non-friend函数才能获得更好的封装性，通过调用class提供的public成员去实现一些工具函数的时候会很有用。

然后为了让这个non-member non-friend的函数显得更自然，作者提供了一种方法，就是将这个函数和对应class放在同一命名空间中：

```cpp
namespace WebBrowserStuff{
  class WebBrowser{ ... };
  void clearBrowser(WebBrowser& wb);
}
```

## 条款24：若所有参数皆需类型转换，请为此采用non-member函数

这点看描述感觉云里雾里的，其实本质也确实很难描述，也许需要通过例子。

书中举了一个有理数的class的例子：

```cpp
class Rational{
public:
    Rational(int numerator=0,int denominator=1);//构造函数刻意不为explicit
																								//允许int-to-Rational
    int numerator()const;
    int denominator()const;
private:
    int numerator;
    int denominator;
}
```

假如使用member函数：

```cpp
class Rational{
public:
	...
	const Rational operator*(const Rational& rhs)const;
};
```

可以解决有理数的相乘：

```cpp
Rational oneEight(1,8);
Rational oneHalf(1,2);
Rational result=oneHalf*oneEight;
result=result*oneEight;
```

但当混入整形时，却会发生错误：

```cpp
result=oneHalf*2;//很好
result=2*oneHalf;//错误
```

因为上面两句代码可以翻译成：

```cpp
result=oneHalf.operator*(2);//很好
result=2.operator*(oneHalf);//错误
```

因为int类型没有重载operator*(const Rational& rhs)，因此报错了。

所以需要实现一个non-member函数：

```cpp
const Rational operator*(const Rational& lhs,const Rational& rhs){
	return ..
}
```

通过这个函数上面两种混合整形的表达式都可以编译通过了，因为会隐式调用Rational的构造函数，相当于：

```cpp
result=operator*(Rational(2), oneHalf);
```

## 条款25：考虑写出一个不抛异常的swap函数

这点主要是为了处理std的默认的swap效率不够高的情况。主要是针对“pimpl手法”的类，即“以指针指向一个对象，内含真正数据”，这种类的swap只需要改变指针指向对象，不需要复制实际数据，因此自己实现特化的swap可以提高效率。

