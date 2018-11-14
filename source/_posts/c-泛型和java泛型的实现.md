---
title: c++泛型(模板)和java泛型的实现
category:
  - c++
tags:
  - c++
  - java
  - 泛型
date: 2018-11-14 11:06:24
---

## 1.前言

面试被问到c++的泛型和java的泛型有什么区别,回答得不是很好,来总结一下.

<!-- more -->

## 2.泛型实现

### (i).c++

c++的泛型实现机制很简单,就是在实际编译时,类似于宏一样,把实际的类型代入模板,并针对不同的类型生成不同的代码,所以编译后代码体积会变大,但执行时就不需要额外的判断了,运行时效率会较高.

所以c++的泛型可以说是以空间换时间.

### (ii).java

java的泛型则比较复杂,java在编译时,它会去执行类型检查和类型推断,然后生成普通的不带泛型的字节码,这种字节码可以被一般的 Java 虚拟机接收并执行,这种技术被称为擦除(erasure).

java编译后不同类型的模板类编译出的是同一份代码.然后在使用时编译器会帮助进行类型转换.

例如:

原代码为:

```java
	Pair<String> pair = new Pair<>("", "");
    pair.setFirst("QMI");
    pair.setSecond("Kang");
    String first = pair.getFirst();
    String second = pair.getSecond();
```

反编译后为:

```java
	Pair pair = new Pair("", "");
    pair.setFirst("QMI");
    pair.setSecond("Kang");
    String first = (String)pair.getFirst();
    String second = (String)pair.getSecond();
```

所以java泛型的实现是在运行时去进行判断和类型转换的,这样会对运行时的效率有一定影响,但编译出来的泛型类的代码只需要一份.

## 3.总结

c++是以泛型类为模板创建出各种不同的实例,而java只是在类上填入Object,运行时进行类型转换.所以c++的泛型是真泛型,java的泛型是伪泛型.