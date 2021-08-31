---
title: LeetCode-重复的DNA序列
category:
  - LeetCode
tags:
  - LeetCode
  - 算法
date: 2021-08-27 11:57:21
---

最近刷到LeetCode的187题，重复的DNA序列，觉得有点意思，记录一下解题过程~
<!-- more -->

## 1.题目

所有 DNA 都由一系列缩写为 `'A'`，`'C'`，`'G'` 和 `'T'` 的核苷酸组成，例如：`"ACGAATTCCG"`。在研究 DNA 时，识别 DNA 中的重复序列有时会对研究非常有帮助。

编写一个函数来找出所有目标子串，目标子串的长度为 10，且在 DNA 字符串 `s` 中出现次数超过一次。

* 示例：

```
  输入：s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT"
  输出：["AAAAACCCCC","CCCCCAAAAA"]
```

## 2.思路

### i.对比首字母相同位置

一开始我想着节省效率，于是想着只记录4中字母的位置，按位置用string的compare函数对比相同首字符后长度为10的串，若和之前某个位置的字符串比对上了，则加入结果中。形成代码大致如下：

``` cpp
#define SEQ_SIZE 10
 public:
  vector<string> findRepeatedDnaSequences(string s)
  {
    vector<string> vecResult;
    if (s.size() <= SEQ_SIZE)
    {
      return vecResult;
    }
    unordered_map<char, vector<int>> mapChar2Poses;
    set<string> result;

    for (int pos = 0; pos <= s.size() - SEQ_SIZE; pos++)
    {
      bool bFound = false;
      vector<int>& vecPoses = mapChar2Poses[s[pos]];
      for (auto prePos : vecPoses)
      {
        if (s.compare(prePos, SEQ_SIZE, s, pos, SEQ_SIZE) == 0)
        {
          result.emplace(s.substr(prePos, SEQ_SIZE));
          bFound = true;
          break;
        }
      }
      if (bFound == false)
      {
        vecPoses.emplace_back(pos);
      }
    }

    for (auto& str : result)
    {
      vecResult.emplace_back(str);
    }
    return vecResult;
  }
```

结果发现compare
