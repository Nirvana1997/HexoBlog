---
title: folly-dynamic
category:
  - c++
tags:
  - folly
date: 2022-01-09 15:33:49
---

今天来看下folly库中的dynamic工具类。这个类的目的是想实现c++的动态类型，类似python、lua那样。之前也大致看了lua弱类型的实现，这次可以来比较下。
<!-- more -->

## 1.使用方法

下面是个官方的例子。

当你使用 `using folly::dynamic;`后，你可以像下面这样使用dynamic：

```cpp
    dynamic twelve = 12; // creates a dynamic that holds an integer
    dynamic str = "string"; // yep, this one is an fbstring

    // A few other types.
    dynamic nul = nullptr;
    dynamic boolean = false;

    // Arrays can be initialized with dynamic::array.
    dynamic array = dynamic::array("array ", "of ", 4, " elements");
    assert(array.size() == 4);
    dynamic emptyArray = dynamic::array;
    assert(emptyArray.empty());

    // Maps from dynamics to dynamics are called objects.  The
    // dynamic::object constant is how you make an empty map from dynamics
    // to dynamics.
    dynamic map = dynamic::object;
    map["something"] = 12;
    map["another_something"] = map["something"] * 2;

    // Dynamic objects may be initialized this way
    dynamic map2 = dynamic::object("something", 12)("another_something", 24);
```

可以看到基本常用的类型都有支持到，能满足大多数场景的使用。

## 2.实现方式

### i.初始化

每次赋值其实是隐式地调用了dynamic对应的构造函数：

####string

```cpp
/*
   * String compatibility constructors.
   */
  /* implicit */ dynamic(std::nullptr_t);
  /* implicit */ dynamic(char const* val);
  /* implicit */ dynamic(std::string val);
  template <
      typename Stringish,
      typename = std::enable_if_t<
          is_detected_v<dynamic_detail::detect_construct_string, Stringish>>>
  /* implicit */ dynamic(Stringish&& s);
```

以上是针对string类型的数据的构造函数，比较特别的是最后声明了一个模板构造函数，传入一个Stringish对象（可以字符串化的对象），需要有data和size对象，用于构造string。

```cpp
template <typename Stringish, typename>
inline dynamic::dynamic(Stringish&& s) : type_(STRING) {
  new (&u_.string) std::string(s.data(), s.size());
}
```

#### Object

```cpp
  /*
   * This is part of the plumbing for array() and object(), above.
   * Used to create a new array or object dynamic.
   */
  /* implicit */ dynamic(void (*)(EmptyArrayTag));
  /* implicit */ dynamic(ObjectMaker (*)());
  /* implicit */ dynamic(ObjectMaker const&) = delete;
  /* implicit */ dynamic(ObjectMaker&&);
```
```cpp
  /*
   * Constructors for integral and float types.
   * Other types are SFINAEd out with NumericTypeHelper.
   */
  template <class T, class NumericType = typename NumericTypeHelper<T>::type>
  /* implicit */ dynamic(T t);
```
```cpp
  /*
   * If v is vector<bool>, v[idx] is a proxy object implicitly convertible to
   * bool. Calling a function f(dynamic) with f(v[idx]) would require a double
   * implicit conversion (reference -> bool -> dynamic) which is not allowed,
   * hence we explicitly accept the reference proxy.
   */
  /* implicit */ dynamic(std::vector<bool>::reference val);
  /* implicit */ dynamic(VectorBoolConstRefCtorType val);

  /*
   * Create a dynamic that is an array of the values from the supplied
   * iterator range.
   */
  template <class Iterator>
  explicit dynamic(Iterator first, Iterator last);
```



### 参考资料

* [facebook/folly](https://github.com/facebook/folly)
