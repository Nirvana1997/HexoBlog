---
title: CMake输出简化
category:
  - CMake
tags:
  - c++
  - CMake
date: 2022-08-02 20:22:39
---

用过Ninja的静默编译模式后，感觉我们原来的CMake系统编译时的输出实在太多了，导致编译时看报错挺不方便的，在读取和处理编译日志时也容易因为日志过大产生问题。因此这次给CMake编译输出做了个“瘦身”。
<!-- more -->

## 1.原编译输出

原来的编译输出大概长这样：

![origin](origin.png)

可以看到有很多“Entering directory”、“Leaving directory”和“Building CXX object”的输出非常多，当工程量比较大时，这些输出其实是比较没用的。

## 2.去除“Building CXX object”日志

```cmake
set_property(GLOBAL PROPERTY RULE_MESSAGES OFF)
```

将`RULE_MESSAGES`设置为off以后，就可以屏蔽这些被编译文件的日志了：

![building](building.png)

最后把这个这个开关使用环境变量控制并输出个启用的日志（顺便记录下cmake中颜色输出的函数）：

```cmake
if(NOT WIN32)
  string(ASCII 27 Esc)
  set(ColourReset "${Esc}[m")
  set(ColourBold  "${Esc}[1m")
  set(Red         "${Esc}[31m")
  set(Green       "${Esc}[32m")
  set(Yellow      "${Esc}[33m")
  set(Blue        "${Esc}[34m")
  set(Magenta     "${Esc}[35m")
  set(Cyan        "${Esc}[36m")
  set(White       "${Esc}[37m")
  set(BoldRed     "${Esc}[1;31m")
  set(BoldGreen   "${Esc}[1;32m")
  set(BoldYellow  "${Esc}[1;33m")
  set(BoldBlue    "${Esc}[1;34m")
  set(BoldMagenta "${Esc}[1;35m")
  set(BoldCyan    "${Esc}[1;36m")
  set(BoldWhite   "${Esc}[1;37m")
endif()

if($ENV{CMAKE_SLIENT} MATCHES "on")
  message("${BoldMagenta}slient mode${ColourReset}")
  set_property(GLOBAL PROPERTY RULE_MESSAGES OFF)
endif()
```

![print](print.png)

## 3.去除“directory”相关日志

```bash
> make -j64 --no-print-directory
```

在make时，加上`--no-print-directory`参数，便可以把“directory”相关的日志屏蔽了：

![directory](directory.png)

可以把这个参数加进环境变量`MAKEFLAGS`中，就可以不用每次加了：

```bash
export MAKEFLAGS='--no-print-directory'
```

## 4.报错染色

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color=always")
```

cmake默认的报错警告等日志是没有颜色的，可以加上`-fdiagnostics-color=always`参数，error就会带颜色了：

![error](error.png)

### 参考资料

* [How to customize cmake output](https://stackoverflow.com/questions/9765547/how-to-customize-cmake-output)

* [cmake: How to suppress “Entering directory” messages?](https://www.mmbyte.com/article/1723.html)
* [Color output of compiler warnings and errors show incorrectly in Output panel and are not parsed for Problems panel](https://github.com/microsoft/vscode-cmake-tools/issues/2278)
* [如何使用cmake获得彩色输出？](https://www.thinbug.com/q/18968979)