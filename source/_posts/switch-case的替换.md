---
title: switch-case的替换
category:
  - 代码风格
tags:
  - switch-case
date: 2020-12-21 23:09:35
---

## 1.前言

switch-case是我们常用的一种语法，几乎所有的语言都有这种语法，可以根据变量的不同情况进行对应处理。但switch-case仅支持整型（int），字符（char）和枚举(enum)，而且switch-case实现类似multi-if，在情况较多时效率较低，并且代码可读性会降低，所以这次想思考下如何优化。

<!-- more -->

## 2.优化方式

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

## 3.性能对比

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



```
"map:1711.493000ms"
"switch:285.326000ms"
"vector:1442.688000ms"

```

