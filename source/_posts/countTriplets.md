title:  Count Triplets That Can Form Two Arrays of Equal XOR
date: 2020-5-10
tags: [Leetcode,周赛]
categories: algorithm

description: 形成两个异或相等数组的三元组数目
---

## Problem description

https://leetcode-cn.com/problems/count-triplets-that-can-form-two-arrays-of-equal-xor/

## Solution

做这道题出发方向走偏了，往滑动窗口的方向越走越远了。既然 xor而且两端区间的 xor 值a == b，那么 a xor b == 0， 区间（i，k）中间的任意值均可作为 j，那么这时的满足的三元组为 k - i, 这样时间复杂度由O(N^4)降为了 O(N^2)

## Code

```kotl
class Solution {
    fun countTriplets(arr: IntArray): Int {
        var sum = 0
        for (i in 0 until arr.size - 1) {
            var a = arr[i]
            for (k in i + 1 until arr.size) {
                a = a xor arr[k]
                if (a == 0) {
                    sum += k - i
                }
            }
        }
        return sum
    }
}
```