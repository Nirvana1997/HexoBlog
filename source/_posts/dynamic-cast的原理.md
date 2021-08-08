---
title: dynamic_cast的原理
category:
  - c++
tags:
  - c++
date: 2021-08-01 23:51:47
---

这次想研究下dynamic-cast的原理。
<!-- more -->

## 1.typeid

[《Type Conversions》](http://www.cplusplus.com/doc/tutorial/typecasting/)上次研究类型转换时的这篇文档最后还讲到了一个运算符——typeid，使用这个运算符，会返回一个type_info对象的引用，这个对象的结构大致如下：

```c++
class type_info {
    // data
public:
    virtual ˜type_info(); //is polymorphic
    bool operator==(const type_info&) const noexcept; // can be compared
    bool operator!=(const type_info&) const noexcept;
    bool before(const type_info&) const noexcept; // ordering
    size_t hash_code() const noexcept; // for use by unordered_map and the like
    const char* name() const noexcept; // name of type
    type_info(const type_info&) = delete; // prevent copying
    type_info& operator=(const type_info&) = delete; // prevent copying
};
```

这个对象包含了一个对象的类型信息。

有趣的是，当typeid作用于指针时，会返回指针自身的类型，但当作用于派生类（有虚函数）的对象或引用时，则会返回这个对象实际的类型。

下面是测试代码：

现在有基类SceneEntry和派生类SceneUser

```c++
class SceneEntry
 {
  public:
   SceneEntry(unsigned long long id, const std::string& name);
   virtual ~SceneEntry() = default;
 };

 class SceneUser : public SceneEntry
 {
  public:
   SceneUser(unsigned long long id, const std::string& name);
   virtual ~SceneUser() = default;
 };

...
  
int main()
{
  SceneEntry* entry = new SceneUser(1, "xiaoming");
  SceneUser user(1,"");
  SceneEntry& entry2 = user;
  std::cout << typeid(entry).name() << std::endl;
  std::cout << typeid(*entry).name() << std::endl;
  std::cout << typeid(entry2).name() << std::endl;
  return 0;
}
```

以上代码可以得到以下输出：

```bash
> ./test
P10SceneEntry
9SceneUser
9SceneUser
```

可以看到对于指针类型，他返回的就是指针声明时的类型，上面的P就表示指针，而引用和类返回的则是他们实际对应的派生类。不过这种获取实际派生类时依赖的是虚函数表，因此没有虚函数的时候也就无从知晓了。

当去掉虚函数以后，输出就变为了：

```bash
P10SceneEntry
10SceneEntry
10SceneEntry
```

最终输出的都是基类。

## 2.dynamic_cast

通过了解type_info，也就基本知道dynamic_cast的原理了，其实它就是利用type_info去判断一个指针所指的类实际是什么的，不过这也依赖于虚函数表，所以当你dynamic_cast一个没有虚函数的类时就会报错（static_cast不会受影响）：

```bash
error: 'SceneEntry' is not polymorphic
```

