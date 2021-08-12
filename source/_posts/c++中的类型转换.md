---
title: c++中dynamic_cast
category:
  - c++
tags:
  - c++
date: 2021-07-24 18:57:55
---

最近项目为了效率，将一些地方的dynamic_cast优化成static_cast，这次正好复习下c++中的类型转换。
<!-- more -->

## 1.c++的类型转换

### i.隐式类型转换

```cpp
short a = 2000;
int b;
b = a;
```

当用short类型的a给int类型的b赋值时，便会自动将short类型转换成int。这种从小类型的整形往更大的整形的转换，就是一次“提升”（promotion），从整形到浮点类型的转化也是一样。个人理解，“提升”就是数值不会溢出转换，这种情况下c++会自动隐式类型转换，并以相同的数值转换过去。

```cpp
class A {};

class B {
public:
  // conversion from A (constructor):
  B (const A& x) {}
  // conversion from A (assignment):
  B& operator= (const A& x) {return *this;}
  // conversion to A (type-cast operator)
  operator A() {return A();}
};

int main ()
{
  A foo;
  B bar = foo;    // calls constructor
  bar = foo;      // calls assignment
  foo = bar;      // calls type-cast operator
  return 0;
}
```

类通过重载赋值运算符，也可以实现隐式转换。

```cpp
void fn(B bar){}

int main()
{
  A foo;
  fn(foo);
}
```

在传入函数时，若只声明了B类型参数的函数，也会发生隐式类型转换。

```cpp
class B {
public:
  explicit B (const A& x) {}
  ...
};

int main ()
{
  A foo;
  B bar (foo);
  bar = foo;
  foo = bar;
  
//  fn (foo);  // not allowed for explicit ctor.
  fn (bar);  

  return 0;
}
```

但当转换的函数使用explicit关键词时，传参是就不能隐式转换了，需要显式地使用=符号或者构造函数的形式转换。

###ii.显式类型转换

```cpp
A* pFoo = new A();
B* pBar = (B*)pFoo;
```

(type)param，这是c中传统的类型转换，不过这种方式直接将A类的指针转换成了完全没有关系的B类的指针，但这种形式的转换在编译时并不会检查这次转换的合法性。为了防止这种不安全的类型转换，c++中新增了4种转换符。

显式类型转换有4种：

####(一).dynamic_cast

dynamic_cast可以用于指针和引用的转换，它会保证本次转换的可靠性。

当从派生类指针转换到基类指针时，由于派生类一定是一个完成的基类，因此dynamic_cast不会做特别的检查。

但当使用它将基类转换为派生类时，它会根据RTTI去判断该基类指针是否可以作为一个完整的目标类。若本次转换不合法，则返回一个空指针。若引用转换不合法，则会抛出bad_cast的异常。

#### (二).static_cast

static_cast可以将指针或引用转换成它的基类或者派生类，当像下面一样转换成毫不相关的类指针时，编译就会报错。

```cpp
  A* foo = new A();
  // error: static_cast from 'A *' to 'B *', which are not related by inheritance, is not allowed
  B* bar = static_cast<B*>(foo);
```

但当转换的类有继承关系时，static_cast便会直接转换，可能会导致错误的转换，引起程序的错误，所以使用static_cast时，程序的安全性需要由程序员来保证。但也正是因为static_cast在运行时不做额外的检查，所以相比dynamic_cast，static_cast的效率更高。

```cpp
Base* pDerived = new Derived(1, "xiaoming");
int count = 0;
while (std::cin >> count)
  {
    int count2 = 0;
    unsigned long long startTime = getUSec();
    for (count2 = 0; count2 < count; ++count2)
    {
      Derived* pDerived2 = (Derived*)(pDerived);
    }
    unsigned long long endTime = getUSec();
    std::cout << "显式转换,次数:" << count2 << ",耗时:" << endTime - startTime << std::endl;

    unsigned long long startTime2 = getUSec();
    for (count2 = 0; count2 < count; ++count2)
    {
      Derived* pDerived2 = static_cast<Derived*>(pDerived);
    }
    unsigned long long endTime2 = getUSec();
    std::cout << "static_cast,次数:" << count2 << ",耗时:" << endTime2 - startTime2 << std::endl;

    unsigned long long startTime3 = getUSec();
    for (count2 = 0; count2 < count; ++count2)
    {
      Derived* pDerived2 = dynamic_cast<Derived*>(pDerived);
    }
    unsigned long long endTime3 = getUSec();
    std::cout << "dynamic_cast,次数:" << count2 << ",耗时:" << endTime3 - startTime3 << std::endl;
  }
```

通过以上代码，大致得到从基类指针转换为派生类时的性能对比：

``` bash
10000
显式转换,次数:10000,耗时:40
static_cast,次数:10000,耗时:39
dynamic_cast,次数:10000,耗时:554
100000
显式转换,次数:100000,耗时:398
static_cast,次数:100000,耗时:455
dynamic_cast,次数:100000,耗时:4116
10000000
显式转换,次数:10000000,耗时:22841
static_cast,次数:10000000,耗时:20263
dynamic_cast,次数:10000000,耗时:176645
```

可以看到普通的显式转换和static_cast效率相当，但dynamic_cast效率就相对比较低了。

但当从派生类转换为基类时：

```cpp
10000
显式转换,次数:10000,耗时:40
static_cast,次数:10000,耗时:40
dynamic_cast,次数:10000,耗时:40
100000
显式转换,次数:100000,耗时:234
static_cast,次数:100000,耗时:212
dynamic_cast,次数:100000,耗时:206
10000000
显式转换,次数:10000000,耗时:22814
static_cast,次数:10000000,耗时:18660
dynamic_cast,次数:10000000,耗时:18444
```

三种方式效率就相当了。

###(三).reinterpret_cast

reintepret_cast的作用和基础的显示类型转换效果基本一致。它可以将任意相关或者不相关的指针或引用相互转换，甚至可以将指针转换为整型，这个过程不会判断指针的合法性，因此和static_cast一样，使用时也需要程序员去保证指针的合法。

### (四).const_cast

const_cast可以改变一个指针或引用是否const的性质，它可以将一个const的指针转变为非const，也可以将一个非const指针转变为const。

## 4.总结

平时我们项目中常用的基本就是dynamic_cast和static_cast，不过由于dynamic_cast在父类指针转换成子类时，会消耗不少性能去转换类的合法性，所以我们项目中不少dynamic_cast在优化中通过一些标志类型的虚函数保证类的合法性，然后使用static_cast转换从而提升性能。此篇主要参考了官网的[《Type Conversions》](http://www.cplusplus.com/doc/tutorial/typecasting/)，从中也了解到dynamic_cast是通过RTTI来确定类的特性，从而判断转换是否合法的，这个目前我还不是很了解，下次继续研究~

