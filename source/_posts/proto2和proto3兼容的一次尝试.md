---
title: proto2和proto3兼容的一次尝试
category:
  - protobuf
tags:
  - c++
  - protobuf
date: 2021-04-01 17:12:42
---

由于项目可能会需要proto3的某些特性，但是当前项目中使用的是proto2，因此我这次开始了proto2升级proto3的历程。标题是一次尝试，那是十有八九我是失败了(╥╯^╰╥)，但通过这次失败还是总结出了一丢丢结论，故书此篇。

(文章中的proto2版本为**2.6.1**，proto3的版本为**3.15.6**。)

<!-- more -->

## 1.proto2与proto3的主要区别

### I.移除了default和required选项

proto3中不再能指定default值，未赋值则使用默认的“0”值，并去除了required关键字。

这一定程度上影响了proto协议的灵活性，项目中还是有一些字段在使用非0的default值后可以减少代码量，使代码简洁明了。

但是这提高了proto的兼容性和稳定性，项目中不同版本的proto可能会改变default值，假如默认值不是0，使用proto通信的两端版本不一致，很有可能会产生一些难以定位的问题。

### II.字段不指定修饰词默认为singular

proto3中，可以不指定修饰词，这样的字段默认为singular。singular字段的好处在于，在设置为默认值时，在序列化后，该字段是不会占用空间的，这对于性能的提升还是有很大帮助的，此次尝试也主要是想使用这个特性。（网上挺多说法是optional修饰的字段也有这个特性，但我实际使用下来optional的字段设置为默认值以后还是会占用空间的，也许是版本更新过了）

不过singular的字段会导致取到默认值后，无法判断该字段是没有赋值还是赋过了0值，这两者在实际项目中有时是需要区别的。这种情况可以使用optional字段，或是使用一个bool值去标记是否赋值。

### III.proto3的repeated字段默认packed=true

对于没有指定packed=true的字段，处理数据时会保存n条key-value结构；而packed=true的字段在处理数据时，会将int类型的数据打成一个包，存储在一段连续的空间中，不保存键值，因此一定程度上能提高性能。

### IV.其他

除此之外，proto3还加入了Any，OneOf等新特性，移除了groups等语法，不过这些新特性对于正在使用proto2的项目可能没有很多适合的使用场景。

## 2.proto2与proto3的兼容

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

```cpp
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

## 3.实际使用

经过以上的尝试，初步判断方案可行后，我开始了实际操作。

我将项目中默认值占用较多空间的proto结构放入了一个proto3的文件中，其余文件开头都添加了proto2的语法标记，成功使用proto文件生成了c++代码。

但是在编译时，出现了一些问题：

首先，ByteSize()这个方法，在proto3中替换为了ByteSizeLong()，不过3中并没有删除该方法，实现改成了调用ByteSizeLong()，所以代码运行不会有问题，只是编译会报一个警告，这个问题并不严重。

但是，proto3生成的类都使用了final去修饰，不管语法是proto2还是proto3，继承了其中的类在编译中会报错。2中没有这个限制，而且通过proto中的类的子类，在代码中管理会更灵活，因此代码中不少地方定义了proto生成类的子类，导致了大量报错，修改的成本较高，于是这次升级在此以失败告终┭┮﹏┭┮。

最终，采用了以下宏，替换set方法，暂时解决设置默认值占用空间的问题，虽然并不优雅，但是是比较稳的一种解法。

```cpp
#define proto_set(cmd, func, value) \
  {                                 \
    if (cmd.func() != value)        \
    {                               \
      cmd.set_##func(value);        \
    }                               \
  }
```

## 4.尚存疑惑

目前，我不清楚为啥proto3要将生成的类用final修饰？

然后代码中的final都是用宏PROTOBUF_FINAL定义的，所以是不是可能可以通过开关控制？

目前有这些疑问，尚未查到答案，先记着看看未来能不能解决吧~
