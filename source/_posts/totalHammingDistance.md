title:  Total Hamming Distance
date: 2020-2-3
tags: [Leetcode,位运算]
categories: algorithm

description: 　
---

## Problem description

  ### https://leetcode-cn.com/problems/total-hamming-distance/


## Solution

一开始直接暴力 O(n^2)果然超时了, `横向逐个比较行不通，换成竖的试试。` 

2 :  0010

4:   0100

14: 1110

海明码距离指的是两个数字的二进制数对应位不同的数量，竖着理解为 第i位 有 x 个 1，y 个 0，其不同的数量为 x * y, 以第三位为例，分别是 0，0，1。1个 1,两个 0。 其不同的数目为2。

我们需要一个 30位的数组去记录每一位的 1 的数目，因为题目中最大的数为 2^30，时间复杂度为 O(N * lgC) C 为数组中最大的数。空间复杂度为O(lgC)

r对于横向比较走不下去时，可以尝试从竖向出发进行解决问题。类似的还有多链表合并的那道题目

## Code

```java
class TotalHammingDistance {
    fun totalHammingDistance(nums: IntArray): Int {
        if (nums.isEmpty()) {
            return 0
        }
        var res = 0
        var bins = IntArray(30)
        for (num in nums) {
            var n = num;
            var i = 0
            while (n != 0) {
                bins[i++] += (n and 1)
                n = n shr 1
            }
        }
        for (bin in bins) {
            res += bin * (nums.size - bin)
        }
        return res
    }
}
```