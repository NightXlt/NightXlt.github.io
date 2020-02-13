title:  Single Number III
date: 2020-2-2
tags: [Leetcode,位运算]
categories: algorithm

description: 　
---

## Problem description

  ### https://leetcode-cn.com/problems/single-number-iii/


## Solution

参考题解 https://leetcode-cn.com/problems/single-number-iii/solution/zhi-chu-xian-yi-ci-de-shu-zi-iii-by-leetcode/

1. x & （-x）保留 x 最右侧的 1,因为-x 是将 x 所有位进行取反 然后最后一位加一。那么这个一必然是来自于两个只出现一次的数 x,y中的一个
2. 同一个数进行异或两次为 0

## Code

```java
class Solution {
    fun singleNumber(nums: IntArray): IntArray {
        val res = IntArray(2)
        var xorRes = 0
        nums.forEach {
            xorRes = xorRes xor it
        }
        val diff = xorRes and -xorRes
        res[0] = 0
        nums.forEach {
            if (it and diff != 0) {
                res[0] = res[0] xor it //使用异或消除冗余的数，其&上 diff 也不等于=0
            }
        }
        res[1] = xorRes xor res[0]
        return res
    }
}
```