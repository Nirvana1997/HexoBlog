---
title: c++构造和析构函数中的虚函数
category:
  - c++
tags:
  - c++
date: 2021-07-17 19:29:05
---

最近我们遇到了一次虚函数相关的宕机，记录一下~
<!-- more -->

## 1.问题代码

``` c++
class Base
{
    public:
    Base() = default;
    virtual ~Base()
    {
        test();
    }
    
    void test()
    {
        virtualFunc();
    }

    virtual void virtualFunc() = 0;
};

class Derived : public Base
{
    public:
    Derived() = default;
    virtual ~Derived() = default;

    virtual void virtualFunc() override
    {
    }
};

int main()
{
    Derived* pClass = new Derived();
    delete pClass;
    return 0;
}
```

出现的问题大致在这样一段代码中，在析构函数中间接调用了虚函数。运行结果发生了错误：

``` bash
> ./test
libc++abi.dylib: Pure virtual function called!
[1]    93942 abort      ./test
```

报错说调用了纯虚函数。

##2.原因

``` c++
class Base
{
    public:
    Base() = default;
    virtual ~Base()
    {
        virtualFunc();
    }

    virtual void virtualFunc()
    {
        cout << "Base destruct" << endl;
    }
};

class Derived : public Base
{
    public:
    Derived() = default;
    virtual ~Derived() = default;

    virtual void virtualFunc() override
    {
        cout << "Derived destruct" << endl;
    }
};

int main()
{
    Derived* pClass = new Derived();
    delete pClass;
    return 0;
}
```

上面是一段测试代码，将纯虚函数改为了普通虚函数并加上了输出。结果如下：

``` bash
> ./test
Base destruct
```

可以看到程序能成功运行了。但是main函数中创建的虽然是派生类，析构函数中调用的virtualFunc却是Base类的。

所以我们代码宕机的原因，就是因为基类析构函数中间接调用到了某个纯虚函数，从而导致了虚函数调用错误。

## 3.结论

经过查阅，c++在析构派生对象时，会有两步析构函数调用：

1. 首先，调用派生类的析构函数；

2. 然后，调用基类的析构函数，但在此时，c++已经将该对象作为派生类的特征“抹除”了，因此在基类的析构函数中的虚函数只会调用基类的虚函数。

对于构造函数也是一样，会先调用父类的构造函数，此时也是没有作为派生类的特征的。

因此，在构造函数和析构函数中，应该尽量避免调用虚函数。
