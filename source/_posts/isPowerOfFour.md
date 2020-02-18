title:   Power of Four
date: 2020-2-4
tags: [Leetcode,位运算]
categories: algorithm

description: 　
---

## Problem description

  ### https://leetcode-cn.com/problems/power-of-four/


## Solution

四的倍数不可能为负数；四的倍数里面只有一个一，可以通过 n & (n - 1)判断；但是怎么将 2 和 4 的倍数区分开呢？2 的倍数的其二进制位为 1的位置是偶数，比如 2，4；4 的倍数的其二进制位为 1的位置是奇数，比如 3，5；所以可以 & 0xaaaaaaaa == 0，但lint 提示 0xaaaaaaa 不是 Int，咦 a是 1010,那第31位岂不是为 1了，而 Int 的最大值是 2^31 - 1，所以只有Int 的最大值只能第 30 位为 1.故可以换成 5: 0101; & 0x55555555 != 0.

## Code

```java
class Solution {
    fun isPowerOfFour(num: Int): Boolean {
        return num > 0 && (num and (num-1) == 0) && (num and 0x55555555) != 0
    }
}
```