title:   Reverse Bits LCCI
date: 2020-3-1
tags: [Leetcode,位运算]
categories: algorithm

description: 翻转数位
---

## Problem description

## https://leetcode-cn.com/problems/reverse-bits-lcci/

## Solution

用一个长度为 1000 的数组记录该数组内连续1的长度，最终通过计算 该数组相邻元素的最大值。

## Code

```java
class Solution {
    fun reverseBits(num: Int): Int {
        var n = num
        val res = IntArray(1000)
        var index = 0
        var max = 0
        while (n > 0) {
            if (n and 1 == 1)
                res[index]++
            else
                index++
            n = n shr 1
        }

        for (i in 0..index) {
            if (max < res[i] + res[i + 1] + 1)
                max = res[i] + res[i + 1] + 1
        }
        return max
    }
}
```