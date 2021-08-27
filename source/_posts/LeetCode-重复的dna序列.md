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

