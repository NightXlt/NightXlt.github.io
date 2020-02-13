title:  Counting Bits
date: 2020-2-1
tags: [Leetcode,位运算]
categories: algorithm

description: 位运算 + 递推式的运用　
---

## Problem description

  ### https://leetcode-cn.com/problems/counting-bits/


## Solution

参考题解

https://leetcode-cn.com/problems/counting-bits/comments/

res[i] : 数字 i 含有多少个 1，令 n = i & (i - 1), 其中 i & i - 1会将 i 最右侧的 1 置为 0，那 res[i] = res[n] + 少了的一

## Code

```java
class Solution {
     fun countBits(num: Int): IntArray {
        val res = IntArray(num + 1)
        for (i in 1..num) {
            res[i] = res[i and (i - 1)] + 1
        }
        return res
    }
}
```