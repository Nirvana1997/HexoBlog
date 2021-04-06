---
title: proto2和proto3兼容的一次尝试
category:
  - protobuf
tags:
  - c++
  - protobuf
date: 2021-04-01 17:12:42
---

## 1.前言

由于项目可能会需要proto3的某些特性，但是当前项目中使用的是proto2，因此我这次开始了proto2升级proto3的历程。标题是一次尝试，那是十有八九我是失败了(╥╯^╰╥)，但通过这次失败还是总结出了一丢丢结论，故书此篇。

## 2.proto2与proto3的区别

### I.移除了default选项

proto3中不再能指定default值，未赋值则使用默认的“0”值。

这一定程度上影响了proto协议的灵活性，项目中还是有一些字段在使用非0的default值后可以减少代码量，使代码简洁明了。

但是这提高了proto的兼容性和稳定性，项目中不同版本的proto可能会改变default值，假如默认值不是0，使用proto通信的两端版本不一致，很有可能会产生一些难以定位的问题。同时，proto3中，设置某字段值为默认值时，在序列化后，该字段是不会占用空间的，这对于性能的提升还是有很大帮助的，此次尝试也主要是想使用这个特性。

。。。

## 3.proto2与proto3的兼容

由于当前项目本身使用的是proto2，规模较大，不太可能整体换成proto3，因此需要考虑2和3的兼容问题。

首先，我尝试使用proto2的lib去编译proto3生成的代码，在编译时，会有很多方法找不到实现，所以这条最简单的路已经断了。

我又在proto3的环境下，进行了以下一些兼容性的测试：

### I.proto语法

```protobuf
syntax = "proto2";

message RedisItem
{
  optional uint32 num = 1 [ default = 0 ];
  optional string str = 2;
  required uint32 num1 = 3;
  repeated uint32 nums = 4;
}
```

我使用了下常用的字段和关键词简单地测试了下，只要开头添加了'syntax = "proto2";'语句指定解析的语法，就可以使用proto3的lib、proto2的语法，去生成相应的代码了，default、required等的关键词不会产生错误。

### II.编译

```protobuf
syntax = "proto3";

message RedisItemTest
{
  uint32 num = 1;
  string str = 2;
}
```

我新增了一个使用proto3语法的文件。我使用g++进行了简单的编译，编译通过，所以语法上proto2和proto3至少在c++这个语言上是可以共存的。

### III.序列化

由于项目中很多结构已经序列化存入过数据库，因此序列话的兼容性是一定得考虑的。

```protobuf
message RedisItemTest
{
  optional uint32 num = 1;
  optional string str = 2;
}
```

<center>
  (i).proto2
</center>

```protobuf
syntax = "proto3";

message RedisItemTest
{
  uint32 num = 1;
  string str = 2;
}
```

<center>(ii).proto3</center>

```c++
int main() {
  ifstream file;
  char str[100];
  file.open("testdata");
  file >> str;
  
  Cmd::RedisItemTest oTest;
  oTest.ParseFromString(str);

  cout << oTest.ShortDebugString() << endl;
  file.close();
  return 0;
}
```

<center>(iii).读取代码</center>

我先使用proto2的库和(i)中proto2语法文件，将num=1，str="testtest"的RedisItemTest序列化。然后，我使用proto3的库，生成(ii)中定义的proto文件，使用(iii)中的读取代码，得到了以下输出：

```bash
./test
num: 1 str: "testtest"
```

所以可以大致得出结论，proto2与proto3至少在c++上，序列化与反序列化方面是基本可以兼容的（由于只是可行性方面的测试，repeated等字段没有进行细致的测试）。

### IV.结论

目前为止，关于兼容性，大致我们得出了以下结论：

使用proto2的lib兼容proto3生成的代码：

​	编译——❎

使用proto3的lib兼容proto2的文件：

​	proto语法——✅

​	编译——✅

​	序列化——✅

所以目前看来，假如想将项目部分proto2结构升级为proto3，将lib升级为proto3，旧的proto文件使用proto2标记，似乎是个可行的方案。

## 4.实际使用

经过以上的尝试，初步判断方案可行后，我开始了实际操作。

我将项目中占用了由于