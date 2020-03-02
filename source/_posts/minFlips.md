title:   Minimum Flips to Make a OR b Equal to c
date: 2020-2-25
tags: [Leetcode,位运算]
categories: algorithm

description: 逐项右移比较
---

## Problem description

  ### https://leetcode-cn.com/problems/minimum-flips-to-make-a-or-b-equal-to-c/


## Solution

因为要比较的是a | b == c 最小翻转总和，可以逐项右移得到对应 a,b,c 的对应二进制位。令为 bitA,bitB,bitC。

当 bitC 为 0时，需要 bitA 和 bitB 都为 0,或的结果才会为0，凡bit位为 1就需要翻转。那需要翻转的次数为 bitA + bitB。

bitC 位为1 时，只需要 bitA 和 bitB 中有 1 个为 1 即可，当 bitA，bitB有 1 个为 1 时，就不需要翻转，否则两者都为 0时，需要翻转一次。

## Code

```java
class Solution {
    fun minFlips(a: Int, b: Int, c: Int): Int {
        var res = 0
        for (i in 0..31) {
            val  bitA = (a shr i) and 1
            val  bitB = (b shr i) and 1
            val  bitC = (c shr i) and 1
            res += if (bitC == 0) {
                bitA + bitB
            } else {
                if (bitA + bitB > 0) 0 else 1
            }
        }
        return res
    }
}
```