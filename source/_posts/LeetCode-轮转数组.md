---
title: LeetCode-轮转数组
category:
  - LeetCode
tags:
  - LeetCode
date: 2021-11-18 12:09:25
---

前两天做了一道轮转数组的LeetCode，题目虽然简单，但解法挺多的，感觉有点意思，记录并巩固下~
<!-- more -->

## 1.题目

> 给你一个数组，将数组中的元素向右轮转 k 个位置，其中 k 是非负数。
>
> 示例 1:
>
> **输入:** nums = [1,2,3,4,5,6,7], k = 3
> **输出:** [5,6,7,1,2,3,4]
> **解释:**
> 向右轮转 1 步: [7,1,2,3,4,5,6]
> 向右轮转 2 步: [6,7,1,2,3,4,5]
> 向右轮转 3 步: [5,6,7,1,2,3,4]
>
> 示例 2:
>
> **输入：**nums = [-1,-100,3,99], k = 2
> **输出：**[3,99,-1,-100]
> **解释:** 
> 向右轮转 1 步: [99,-1,-100,3]
> 向右轮转 2 步: [3,99,-1,-100]

## 2.思路

### (1).创建新数组

最简单的办法可以直接创建一个新数组来存储旋转后的数据，最后将新数组数据赋值回目标数组。

``` cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> newArr(n);
        for (int i = 0; i < n; ++i) {
            newArr[(i + k) % n] = nums[i];
        }
        nums.assign(newArr.begin(), newArr.end());
    }
};
```

这种做法，时间复杂度是O(n)，空间复杂度是O(n)。因为至少需要遍历一遍，所以时间复杂度O(n)已经没有多少提升空间了，但是理论上只是进行交换的话不需要O(n)的额外空间，空间复杂度上是可以优化的。

### (2).环形交换

原地交换的话，就不需要额外的空间了，只需要将被交换的数据存入一个临时变量，然后再替换他的下一个目标位置就可以，这样额外只需要一个临时变量的空间，空间复杂度就变为了O(1)。但是这种方法的有一个关键点就是如何遍历的问题，因为单从一点出发回到原点，此时可能是有未遍历到的元素的。

官方给了一种数学推导方式去确定如何遍历：

> 由于每次循环最终回到了起点，故该过程恰好走了整数数量的圈，不妨设为 a 圈；再设该过程总共遍历了 b 个元素。因此，我们有 an=bk，即 an 一定为 n,k 的公倍数。又因为我们在第一次回到起点时就结束，因此 a 要尽可能小，故 an 就是 n,k 的最小公倍数 lcm(n,k)，因此 b 就为 lcm(n,k)/k。

总结来说，其实一次循环后，遍历到的元素每个是相隔 lcm(n,k) 的，所以只需要分别以数组前 lcm(n,k) 个元素为起点进行一个上述的循环，就可以遍历完所有元素。可以看下图例子：

![traversal](traversal.png)

知道这个规律后使用一个变量对遍历过的元素计数就行了，从头开始挨个进行循环是不会遍历到重复元素的，所以最终总的遍历过的数目与数组长度相同就可以了。

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k = k % n;
        int count = gcd(k, n);
        for (int start = 0; start < count; ++start) {
            int current = start;
            int prev = nums[start];
            do {
                int next = (current + k) % n;
                swap(nums[next], prev);
                current = next;
            } while (start != current);
        }
    }
};
```

这种方法每个元素仅会遍历一遍，额外空间只需要一些固定的临时变量，所以时间复杂度是O(n)，空间复杂度是O(1)。

### (3).数组翻转

这种方法比较有意思，是通过翻转的方式实现的。

该方法通过整体翻转后再局部翻转的方式实现数组旋转，引用一条评论可以很清楚的看懂思路：

> nums = "----->-->"; k =3
> result = "-->----->";
> 
> reverse "----->-->" we can get "<--<-----"
> reverse "<--" we can get "--><-----"
> reverse "<-----" we can get "-->----->"

知道思路后代码就很容易出来了：

```cpp
class Solution {
public:
    void reverse(vector<int>& nums, int start, int end) {
        while (start < end) {
            swap(nums[start], nums[end]);
            start += 1;
            end -= 1;
        }
    }

    void rotate(vector<int>& nums, int k) {
        k %= nums.size();
        reverse(nums, 0, nums.size() - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, nums.size() - 1);
    }
};
```

## 3.总结

很多题目虽然挺简单，但是能看到很多有意思的思路和方法，感觉之前评论区看少了，今后可以多看看~

### 参考资料

* [轮转数组-力扣](https://leetcode-cn.com/problems/rotate-array/solution/xuan-zhuan-shu-zu-by-leetcode-solution-nipk/)
