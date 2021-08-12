---
title: c++中的拷贝构造函数调用时机
category:
  - c++
tags:
  - c++
  - 拷贝构造函数
date: 2018-08-17 21:53:34
---

在码代码时会经常遇到stl容器类的传递,stl类内数据量较大时,拷贝时会耗费大量资源,需要避免,所以需要了解何时会调用拷贝构造函数.

网上看到一种答案:

> 拷贝构造函数调用的几种情况： 
> 
> 1. 当用类的一个对象去初始化该类的另一个对象（或引用）时系统自动调用拷贝构造函数实现拷贝赋值。 
> 2. 若函数的形参为类对象，调用函数时，实参赋值给形参，系统自动调用拷贝构造函数。 
> 3. 当函数的返回值是类对象时，系统自动调用拷贝构造函数。

为验证正确性,以下做了个实验.

<!-- more -->

## 1.拷贝构造函数调用时机

### (i).代码及输出

为了确认c++中拷贝构造函数的调用时机,执行以下代码:

```cpp
#include <iostream>
#include <vector>

using namespace std;

class Copy{
public:
    int val = 0;
    
    Copy(){
        cout<<"construct"<<endl;
    }
    
    ~Copy(){}
    
    Copy(const Copy& copy){
        cout<<"copy"<<endl;
    }
};

void asParam(Copy copy){
    
}

Copy returnCopy(){
    Copy copy;
    cout<<"  return:"<<endl<<"    ";
    return copy;
}

int main(int argc, const char * argv[]) {
    Copy copy;
    
    cout<<"align:"<<endl<<"  ";
    Copy copy1 = copy; 
       
    cout<<"init:"<<endl<<"  ";
    Copy copy2 = Copy(copy);
    
    cout<<"asParam:"<<endl<<"  ";
    asParam(copy);
    
    cout<<"in container:"<<endl;
    vector<Copy> copys;
    cout<<"  push_back:"<<endl<<"    ";
    copys.push_back(copy);
    cout<<"  copy container:"<<endl<<"    ";
    vector<Copy> copys1 = copys;
    
    cout<<"returnCopy:"<<endl<<"  ";
    returnCopy();
    
}
```

得到输出:

```
construct
align:
  copy
init:
  copy
asParam:
  copy
in container:
  push_back:
    copy
  copy container:
    copy
returnCopy
  construct
  return:
```

### (ii).结论

由以上输出,再对照网上找到的答案:

> 1. 当用类的一个对象去初始化该类的另一个对象（或引用）时系统自动调用拷贝构造函数实现拷贝赋值。 
> 2. 若函数的形参为类对象，调用函数时，实参赋值给形参，系统自动调用拷贝构造函数。 
> 3. 当函数的返回值是类对象时，系统自动调用拷贝构造函数。

可以得到:

1. 用一个对象初始化时会调用拷贝构造函数,包括用"="赋值和Copy(copy)这样用类作为构造函数的参数的初始化.
2. 作为形参时,也会调用到拷贝构造函数.
3. 在作为函数的返回值时,**并不会调用拷贝构造函数**.

## 2.类作为函数的返回值

可以看到在作为函数返回值时,并不会调用拷贝构造函数,但局部的对象在栈中,函数结束后会随之销毁,不可能返回函数中的局部对象.为此,用以下代码再次实验:

### (i).代码及输出

```cpp
#include <iostream>
#include <vector>

using namespace std;

class Copy{
public:
    int val = 0;
    
    Copy(){
        cout<<"construct"<<endl;
    }
    
    Copy(int i):val(i){}
    
    ~Copy(){}
    
    Copy(const Copy& copy){
        cout<<"copy"<<endl;
    }
};

Copy returnCopy(){
    Copy copy(5);
    cout<<"local: val "<<copy.val<<" ,address "<<&copy<<endl;
    return copy;
}

int main(int argc, const char * argv[]) {
    Copy copy = returnCopy();
    cout<<"return: val "<<copy.val<<" ,address "<<&copy<<endl;
}
```

输出:

```
local: val 5 ,address 0x7ffeefbff578
return: val 5 ,address 0x7ffeefbff578
```

### (ii).结论

可以看到,很神奇的是,返回的居然是局部对象,有点超出我的理解了.

然后经过资料查阅,发现结论是这样的:

按照正常情况在return时是会发生一次拷贝,但是g++会有一种RVO(return value optimization)的技术,在编译时会发生[Copy elision](https://en.wikipedia.org/wiki/Copy_elision#Return_value_optimization)([复制省略](https://zh.wikipedia.org/wiki/%E5%A4%8D%E5%88%B6%E7%9C%81%E7%95%A5)).

大致意思就是避免不必要的拷贝,如这里,使用临时变量为同类型的变量赋值时,转换为直接初始化复制到的变量,因此这里不会调用到拷贝构造函数.

## 3.总结

c++相比java等语言少了一层jvm,本身语言更偏底层了,但很多内存的管理分配还是值得深究.

而且通过这次发现编译器功能真的十分强大,能给我们的程序带来很多意想不到的优化.但这样的同时,也会给我们的程序带来更多变数,需要程序员更深入了解c++与编译器的一些优化策略,从而更好地控制我们的代码与程序.

未来仍需努力~
