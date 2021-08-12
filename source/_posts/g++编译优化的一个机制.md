---
title: g++编译优化的一个机制
category:
  - c++
tags:
  - 编译器
  - c++
date: 2021-07-03 18:18:37
---

最近工作中同事在研究volatile关键字的时候，遇到了个有趣的情况，和g++的编译优化有关，在这里记录一下~
<!-- more -->

## 1.遇到问题

这次遇到的问题是这样的，我们使用这样一段代码测试volatile的特性：

```cpp
#include <pthread.h>
#include <stdio.h>

#ifdef _VOLATILE
volatile bool b = true;
#else
bool b = true;
#endif

void* test(void* p)
{
    b = false;
    return 0;
}

int main(){
    pthread_t thread_id;
    pthread_create(&thread_id, NULL, &test, NULL);
    while (b)
    {
    }
    return 0;
}
```

 尝试了该段代码在变量b是否声明volatile，是否启用O2编译优化的几种情况下的执行结果。

结果发现，在未声明volatile、启用O2的情况下，程序卡死了，但其他情况均可以正常退出。

## 2.原因分析

没想明白为啥，于是查看了汇编代码，发现问题在于对while(b)这行代码的优化。

非volatile，无O2的while(b)：

``` assembly
				...
LBB1_1:                                 ## =>This Inner Loop Header: Depth=1
        testb   $1, _b(%rip)
        je      LBB1_3
## %bb.2:                               ##   in Loop: Header=BB1_1 Depth=1
        jmp     LBB1_1
LBB1_3:
        ...
```

非volatile，开启O2优化的while(b)：

``` assembly
				...
				cmpb    $0, _b(%rip)
        je      LBB1_2
        .p2align        4, 0x90
LBB1_1:                                 ## =>This Inner Loop Header: Depth=1
        jmp     LBB1_1
LBB1_2:
				...
```

可以看到非O2的情况下，b的判断是在每次循环都会判断的，而O2的优化中，由于b只在子线程中修改，所以编译器判定b不会变化，仅比较了一次，然后开始死循环了。

而b声明为volatile以后，编译器在优化时不会把判断的流程优化掉，因此不会卡死。

## 3.总结

经过这次主要得出的结论：

1.O2优化下，会将主线程中不会变化的值优化，当做类似常量来处理；

2.volatile在多线程下可以防止这种优化带来的问题。

