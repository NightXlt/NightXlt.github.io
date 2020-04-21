title:   Minimum Number of Frogs Croaking
date: 2020-4-19
tags: [Leetcode,贪心]
categories: algorithm

description: 数青蛙
---

## Problem description

https://leetcode-cn.com/problems/minimum-number-of-frogs-croaking/

## Solution

每个 croak 中 c 和 k 之间出现了多少次c 表示，至少有多少只青蛙。如 `cccroaroaroa`kkk,当k还未出现时，前面有3 个 c，0 个 k 表示至少有 3 次青蛙。代码体现就是统计 c,k 的出现次数并求出其最大差值。此外还需要保证整个统计频次过程中 
$$
f_c \ge f_r \ge f_o \ge f_a \ge f_k （f_c：表示 c 出现频次）
$$
并且在最后保证
$$
f_c == f_r == f_o == f_a == f_k
$$

## Code

```java
import kotlin.math.max

class Solution {
fun minNumberOfFrogs(croakOfFrogs: String): Int {
    if (croakOfFrogs.isEmpty()) return -1
    val count = Array(5) { 0 }
    var res = 0
    for (c in croakOfFrogs) {
        when (c) {
            'c' -> count[0]++
            'r' -> count[1]++
            'o' -> count[2]++
            'a' -> count[3]++
            'k' -> count[4]++
        }
        res = max(count[0] - count[4], res)
        if (count[0] < count[1] || count[1] < count[2] || count[2] < count[3] || count[3] < count[4]) return -1
    }
    if (count.all { it == count[0] }) return res
    return -1
}
}
```