title:   Beautiful Arrangement II
date: 2020-3-25
tags: [Leetcode,数学]
categories: algorithm

description: 排列题目
---

## Problem description

https://leetcode-cn.com/problems/beautiful-arrangement-ii/

## Solution

我以为自己在做回溯题，没想竟然是道数学题。。。

参考： https://leetcode-cn.com/problems/beautiful-arrangement-ii/solution/java-bu-duan-fan-zhuan-zai-tong-guo-guan-cha-zhi-j/

可以观察得出规律：前 K 个数字 偶数下标是从 1 递增，奇数下标从 n 递减。 从 K 个数字开始，根据 K的奇偶进行递增亦或递减。

## Code

```java
class Solution {
 
    fun constructArray(n: Int, k: Int): IntArray {
        val res = IntArray(n)
        var odd = 1
        var even = n
        for (i in 0 until k) {
            if (i and 1 == 1) {
                res[i] = even--
            } else {
                res[i] = odd++
            }
        }
        if (k and 1 == 1) {
            for (i in k until n) {
                res[i] = res[i - 1] + 1
            }
        } else {
            for (i in k until n) {
                res[i] = res[i - 1] - 1
            }
        }
        return res
    }
}
```