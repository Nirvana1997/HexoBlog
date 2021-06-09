---
title: switch-case的替换
category:
  - c++
tags:
  - switch-case
date: 2020-12-21 23:09:35
---

switch-case是我们常用的一种语法，几乎所有的语言都有这种语法，可以根据变量的不同情况进行对应处理。但switch-case仅支持整型（int），字符（char）和枚举(enum)，而且switch-case实现应该是类似multi-if，在情况较多时效率较低，并且代码可读性会降低，所以这次想思考下如何优化。

<!-- more -->

## 1.优化方式

一次switch-case的过程其实就是一次寻找元素的过程。而在元素固定的情况下，寻找元素，最快的结构就是HashTable，也就是c++的unordered_map。只要通过将处理函数抽象成通用函数便可以利用。
like this:

``` c++
enum CASE
{
    CASE_1,
};

struct Param
{
    int param = 0;
};

unordered_map<int, function<void(Param)>> mapFunc;

void handle(CASE eCase, Param param)
{
    auto it = mapFunc.find((int)eCase);
    if (it != mapFunc.end())
    {
        it->second(param);
    }
    else
    {
        cout << "error" << endl;
    }
}

// 注册处理函数
mapFunc[(int)CASE_1] = [](Param param) {
    cout << "handle case 1" << endl;
};
```

这样，就实现了使用unordered_map来完成switch-case的工作。当case较多，处理函数较复杂时，这样可以有效地减少代码的复杂度。并且因为unordered_map使用的是hashtable的树结构，所以相比比switch-case有更好的时间效率（理论上）。

## 2.性能对比

刚才对于性能的对比是基于理论的猜想，实际性能需要数据支持，于是我写了下面的测试代码：

```c++
unordered_map<int, function<void(Param)>> mapFunc =
    {
        {CASE_1, [](Param param) {}},
        {CASE_2, [](Param param) {}},
        {CASE_3, [](Param param) {}},
        {CASE_4, [](Param param) {}},
        {CASE_5, [](Param param) {}},
};

vector<pair<int, function<void(Param)>>> vectorFunc =
    {
        make_pair(CASE_1, [](Param param) {}),
        make_pair(CASE_2, [](Param param) {}),
        make_pair(CASE_3, [](Param param) {}),
        make_pair(CASE_4, [](Param param) {}),
        make_pair(CASE_5, [](Param param) {}),
};

void handleWithMap(CASE eCase, Param param)
{
    auto it = mapFunc.find((int)eCase);
    if (it != mapFunc.end())
    {
        it->second(param);
    }
    else
    {
        cout << "error" << endl;
    }
}

void handleWithVector(CASE eCase, Param param)
{
    auto it = find_if(vectorFunc.begin(), vectorFunc.end(), [&](const pair<int, function<void(Param)>> &func) -> bool {
        return eCase == func.first;
    });
    if (it != vectorFunc.end())
    {
        it->second(param);
    }
    else
    {
        cout << "error" << endl;
    }
}

void handleWithSwitch(CASE eCase, Param param)
{
    switch (eCase)
    {
    case CASE_1:
    {
    }
    break;
    case CASE_2:
    {
    }
    break;
    case CASE_3:
    {
    }
    break;
    case CASE_4:
    {
    }
    break;
    case CASE_5:
    {
    }
    break;
    default:
    {
    }
    break;
    }
}

int main()
{
    int times = 10000000;
    struct timeval start;
    struct timeval end;
    gettimeofday(&start, NULL);
    srand(123);
    for (int i = 0; i < times; i++)
    {
        CASE randCase = static_cast<CASE>(rand() % 5);
        handleWithMap(randCase, Param());
    }
    gettimeofday(&end, NULL);
    double time_use = (end.tv_sec - start.tv_sec) * 1000000 + (end.tv_usec - start.tv_usec);
    cout.setf(ios::fixed);
    cout << "time:" << time_use / 1000 << "ms" << endl;

    gettimeofday(&start, NULL);
    srand(123);
    for (int i = 0; i < times; i++)
    {
        CASE randCase = static_cast<CASE>(rand() % 5);
        handleWithSwitch(randCase, Param());
    }
    gettimeofday(&end, NULL);
    time_use = (end.tv_sec - start.tv_sec) * 1000000 + (end.tv_usec - start.tv_usec);
    cout << "time:" << time_use / 1000 << "ms" << endl;

    gettimeofday(&start, NULL);
    srand(123);
    for (int i = 0; i < times; i++)
    {
        CASE randCase = static_cast<CASE>(rand() % 5);
        handleWithVector(randCase, Param());
    }
    gettimeofday(&end, NULL);
    time_use = (end.tv_sec - start.tv_sec) * 1000000 + (end.tv_usec - start.tv_usec);
    cout << "time:" << time_use / 1000 << "ms" << endl;

    return 0;
}
```

结果有些出乎意料：

```
"map:1711.493000ms"
"switch:285.326000ms"
"vector:1442.688000ms"

```

switch-case的效率和stl相比快了很多，难道switch-case的实现原理不是挨个对比？

于是我又通过测试代码，获取对应汇编：

```c++
int main()
{
	int a, sc = 1;
	switch (sc)
	{
		case 1:
			a = 1;
		break;
		case 2:
			a = 2;
		break;
	}
}
```

```assembly
main:
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 1
        mov     eax, DWORD PTR [rbp-4]
        cmp     eax, 1
        je      .L3
        cmp     eax, 2
        je      .L4
        jmp     .L2
.L3:
        mov     DWORD PTR [rbp-8], 1
        jmp     .L2
.L4:
        mov     DWORD PTR [rbp-8], 2
        nop
.L2:
        mov     eax, 0
        pop     rbp
        ret
```

大概可以看到switch-case就是简单的比较和跳转，时间复杂度理论上是O(n)，但实际上为什么switch-case会比O(1)的unordered_map和O(n)的vector效率相差这么多呢？我用的是O0的优化，使用了5个case，猜想可能和优化等级和case数目有关？于是我使用不同数目的case在O0和O2下做了性能统计。(如何使用变量控制n个case，头疼了蛮久，不太会使用宏去生成，用脚本去生成c++代码又觉得有些浪费，最后网上找到这篇博客：[用C语言宏批量生成代码的思考与实现](https://zhou-yuxin.github.io/articles/2016/%E7%94%A8C%E8%AF%AD%E8%A8%80%E5%AE%8F%E6%89%B9%E9%87%8F%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E7%9A%84%E6%80%9D%E8%80%83%E4%B8%8E%E5%AE%9E%E7%8E%B0/index.html)，使用博主提供的头文件，使用宏生成了测试代码——[switchcaseOptTest](https://github.com/Nirvana1997/TestRepo/tree/master/switchcaseOptTest))

|              | unordered_map | switch-case | vector  | array  |
| ------------ | ------------- | ----------- | ------- | ------ |
| O0，case-5   | 1610ms        | 287ms       | 1393ms  | 584ms  |
| O0，case-100 | 1718ms        | 318ms       | 8346ms  | 622ms  |
| O0，case-255 | 1929ms        | 326ms       | 17491ms | 1278ms |
| O2，case-5   | 229ms         | 60ms        | 181ms   | 200ms  |
| O2，case-100 | 278ms         | 60ms        | 472ms   | 239ms  |
| O2，case-255 | 312ms         | 60ms        | 805ms   | 314ms  |

发现不论在O0还是O2的情况下，switch-case的效率都是最高的，而且在O2的情况下几乎不会随着case数增长而增长；map消耗时间随case数缓慢增长而vector随case数接近线性增长。

这与我们理论上得到的结论大相径庭。(网上找到一篇go中类似的性能对比[map vs switch performance in go](https://stackoverflow.com/questions/46789259/map-vs-switch-performance-in-go)，其中使用slice代替switch-case，效率得到了一定的提升，于是我在测试中加入了array，但实际性能和unordered_map类似）

## 3.switch-case原理

根据[Advantage of switch over if-else statement](https://stackoverflow.com/questions/97987/advantage-of-switch-over-if-else-statement)了解到，通常情况下编译器会为switch-case建立二叉决策树或者建立跳转表，因此和multi-if的遍历的做法其实是不一样的，效率回避multi-if高很多。

但是决策树和跳转表的结构照理和stl的map和unordered_map的结构类似，但是为什么效率还是差了很多呢？我猜测是由于stl的结构占得内存较大，默认的switch-case结构占得内存较小，在cache机制中容易整段进入cache，所以效率较高，但是我也没细究了。

## 4.结论

switch-case作为原生的机制，效率还是非常高的，在百个case这个量级下比stl的容器效率高很多（目前大多项目一个case百量级应该够用了），但switch-case在case较多的情况下，即使每个case只用一个函数可读性感觉还是没那么好，所以在效率没那么敏感，case较多的情况下还是用一些结构如map等去整理case比较好。
