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

#### string

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
以上这些构造函数是为了适配数组、map和一般对象，通过ObjectMaker来方便对应对象的构造。

#### Num

```cpp
  /*
   * Constructors for integral and float types.
   * Other types are SFINAEd out with NumericTypeHelper.
   */
  template <class T, class NumericType = typename NumericTypeHelper<T>::type>
  /* implicit */ dynamic(T t);
```
针对整型和浮点数，使用了一个模板函数和NumericTypeHelper来帮助构造和区分Num的数据。

#### Iterator

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

以上是迭代器的构造函数。

### ii.类型维护

在创建一个dynamic对象后会使用一个成员`type_`来记录类型，在对应构造函数中会初始化该成员：

```cpp
inline dynamic::dynamic() : dynamic(nullptr) {}

inline dynamic::dynamic(std::nullptr_t) : type_(NULLT) {}

inline dynamic::dynamic(void (*)(EmptyArrayTag)) : type_(ARRAY) {
  new (&u_.array) Array();
}

inline dynamic::dynamic(ObjectMaker (*)()) : type_(OBJECT) {
  new (getAddress<ObjectImpl>()) ObjectImpl();
}

inline dynamic::dynamic(char const* s) : type_(STRING) {
  new (&u_.string) std::string(s);
}
...
```

有了`type_`存储类型就可以用更方便地实现方法，同时也因此可以提供类型的接口，方便调用者使用：

```cpp
/*
   * Returns true if this dynamic is of the specified type.
   */
  bool isString() const;
  bool isObject() const;
  bool isBool() const;
  bool isNull() const;
  bool isArray() const;
  bool isDouble() const;
  bool isInt() const;

  /*
   * Returns: isInt() || isDouble().
   */
  bool isNumber() const;

  /*
   * Returns the type of this dynamic.
   */
  Type type() const;
```

### iii.ObjectMaker

ObjectMaker是为了方便构造对象，大致代码如下：

```cpp
// Helper object for creating objects conveniently.  See object and
// the dynamic::dynamic(ObjectMaker&&) ctor.
struct dynamic::ObjectMaker {
  friend struct dynamic;

  explicit ObjectMaker() : val_(dynamic::object) {}
  explicit ObjectMaker(dynamic key, dynamic val) : val_(dynamic::object) {
    val_.insert(std::move(key), std::move(val));
  }

  // Make sure no one tries to save one of these into an lvalue with
  // auto or anything like that.
  ObjectMaker(ObjectMaker&&) = default;
  ObjectMaker(ObjectMaker const&) = delete;
  ObjectMaker& operator=(ObjectMaker const&) = delete;
  ObjectMaker& operator=(ObjectMaker&&) = delete;

  // This returns an rvalue-reference instead of an lvalue-reference
  // to allow constructs like this to moved instead of copied:
  //  dynamic a = dynamic::object("a", "b")("c", "d")
  ObjectMaker&& operator()(dynamic key, dynamic val) {
    val_.insert(std::move(key), std::move(val));
    return std::move(*this);
  }

 private:
  dynamic val_;
};
```

比较特别的是其中的构造函数仅提供了const引用和右值的形式。使用const引用可以最小化权限，避免一些不必要的修改引起的错误。使用右值形式可以避免拷贝，提高性能。

### iv.NumericTypeHelper

NumericTypeHelper这个类是folly为了方便处理整型和浮点数这两类Num类型的数据。

dynamic中将整型统一使用int64类型，浮点型统一使用double类型，会通过以下的机制去区分：

```cpp
template <
    class T,
    class NumericType /* = typename NumericTypeHelper<T>::type */>
dynamic::dynamic(T t) {
  type_ = TypeInfo<NumericType>::type;
  new (getAddress<NumericType>()) NumericType(NumericType(t));
}
```

可以看到构造时会使用`TypeInfo<NumericType>::type`作为type，默认情况下则是使用`typename NumericTypeHelper<T>`中生成的type。NumericTypeHelper针对整型、bool、float、double，会生成不同的type：

```cpp
template <class T>
struct dynamic::NumericTypeHelper<
    T,
    typename std::enable_if<std::is_integral<T>::value>::type> {
  static_assert(
      !kIsObjC || sizeof(T) > sizeof(char),
      "char-sized types are ambiguous in objc; cast to bool or wider type");
  using type = int64_t;
};
template <>
struct dynamic::NumericTypeHelper<bool> {
  using type = bool;
};
template <>
struct dynamic::NumericTypeHelper<float> {
  using type = double;
};
template <>
struct dynamic::NumericTypeHelper<double> {
  using type = double;
};
```

生成对应NumericTypeHelper后就在构造函数中使用TypeInfo去取到类型，TypeInfo也是上面定义的一个类似的struct，可以根据数据的type获取对应的dynamic中的type：

```cpp
#define FOLLY_DYNAMIC_DEC_TYPEINFO(T, val)     \
  template <>                                  \
  struct dynamic::TypeInfo<T> {                \
    static const char* const name;             \
    static constexpr dynamic::Type type = val; \
  };                                           \
  //

FOLLY_DYNAMIC_DEC_TYPEINFO(std::nullptr_t, dynamic::NULLT)
FOLLY_DYNAMIC_DEC_TYPEINFO(bool, dynamic::BOOL)
FOLLY_DYNAMIC_DEC_TYPEINFO(std::string, dynamic::STRING)
FOLLY_DYNAMIC_DEC_TYPEINFO(dynamic::Array, dynamic::ARRAY)
FOLLY_DYNAMIC_DEC_TYPEINFO(double, dynamic::DOUBLE)
FOLLY_DYNAMIC_DEC_TYPEINFO(int64_t, dynamic::INT64)
FOLLY_DYNAMIC_DEC_TYPEINFO(dynamic::ObjectImpl, dynamic::OBJECT)
```

## 3.总结

folly的dynamic实现还是相当复杂的，代码中有非常多对模板和SFINAE的使用，看起来还是蛮吃力的，不过通过这次也学到了不少，下次写代码的时候尝试着多去用用看。总的来说，folly的dynamic的实现和Lua的动态类型很像，在初始化时赋予类型，这样使用时会比boost::any和或std::any这种通过类型擦除实现的方便很多。

### 参考资料

* [facebook/folly](https://github.com/facebook/folly)
