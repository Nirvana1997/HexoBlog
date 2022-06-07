---
title: lua与c++的交互
category:
  - c++
tags:
  - c++
  - lua
date: 2018-08-22 10:08:27
---

Lua作为一个运行效率非常高的脚本语言,简单方便,如今很多游戏开发都会用到.今天研究下c++和lua是如何交互的~

<!-- more -->

## 1.c++调用lua

我是在macos上实验的,过程应该和linux上类似.使用的lua版本是5.3.0.

c\+\+调用lua原理主要是通过Lua的堆栈,一方将传递的参数以及参数个数压入栈中,另一方则在栈中取出参数,并将返回值压入栈中,由另一方取出,实现交互.这个过程和c\+\+和汇编(如nasm)的交互过程很像.

### (i).添加依赖

将liblua.a添加到项目中:

![lib](lib.png)

设置search paths:

Header Search Paths设置为lua安装位置,用来搜索头文件.
	
Library Search Paths设置为项目存放.a库的目录.

![lib](path.png)

### (ii).代码测试

```cpp
#include <iostream>

extern "C" {
#include "lua.h"
#include "lualib.h"
#include "lauxlib.h"
}

using namespace std;

int main()
{
    //1.创建一个state
    lua_State *L = luaL_newstate();
    //2.入栈操作
    lua_pushstring(L, "Hello World~");
    //3.取值操作
    if (lua_isstring(L, 1)) { //判断是否可以转为string
        cout << lua_tostring(L, 1) << endl; //转为string并返回
    }
    //4.关闭state
    lua_close(L);
}
```

### (iii).运行代码

运行结果:

![lib](run.png)

### (iv).求和代码:

lua代码:

``` lua
function Sum(a, b)
  return (a+b);
end
```

c++代码:

```cpp
#include <iostream>

extern "C" {
#include "lua.h"
#include "lualib.h"
#include "lauxlib.h"
}

using namespace std;

int main()
{
    //创建Lua状态
    lua_State *L = luaL_newstate();
    if (L == NULL)
    {
        return 0;
    }
    
    //加载Lua文件
    int bRet = luaL_loadfile(L, "sum.lua");
    if (bRet)
    {
        cout << "load file error" << endl;
        return 0;
    }
    
    //运行Lua文件
    bRet = lua_pcall(L, 0, 0, 0);
    if (bRet)
    {
        cout << "pcall error" << endl;
        return 0;
    }
    //读取函数
    lua_getglobal(L, "Sum");        // 获取函数，压入栈中
    lua_pushinteger(L, 10);          // 压入参数
    lua_pushinteger(L, 8);          // 压入参数
    int iRet = lua_pcall(L, 2, 1, 0);// 调用函数，调用完成以后，会将返回值压入栈中，第一个2表示参数个数，第二个1表示返回结果个数。
    if (iRet)                       // 调用出错
    {
        const char *pErrorMsg = lua_tostring(L, -1);
        cout << pErrorMsg << endl;
        lua_close(L);
        return 0;
    }
    if (lua_isinteger(L, -1))        //取值输出
    {
        int sum = lua_tointeger(L, -1);
        cout << sum << endl;
    }
    
    //关闭state
    lua_close(L);
    return 0;
}

```

结果:

![sum](sum.png)

## 2.lua调用c++

lua调用c\+\+原理主要是将c\+\+代码编译成.so动态库,然后在lua中用require引入,从而调用.通信还是通过lua栈来实现的.这里就不展开了.具体可以参考[lua官方文档](http://www.lua.org/manual/),还有[lua5.3中文文档](http://cloudwu.github.io/lua53doc/)

## 3.LuaTinker

以上介绍的都是lua官方的提供给c的一些api,较为底层,其实有很多c++和lua代码的绑定库,将以上的栈操作以及函数注册等封装了起来,如LuaBind,LuaTinker等,我们的项目中使用的就是LuaTinker.LuaTinker使用方法可以在[git仓库](https://github.com/zupet/LuaTinker)中查看.
