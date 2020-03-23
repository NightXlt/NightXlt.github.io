title:    Count Numbers with Unique Digits
date: 2020-3-23
tags: [Leetcode,动态规划]
categories: algorithm

description: 计算各个位数不同的数字个数
---

## Problem description

https://leetcode-cn.com/problems/count-numbers-with-unique-digits/

## Solution

Emmm，想了半天，这不是回溯题呀。LT 的分类真的烂。计算出出所有位数都不相等的数字的个数，假设有一个 n 位的数字，我们已经知道前 n - 1位的不相等的数字个数，那么n 位unique数字 dp[n] = dp[n- 1] + (dp[n - 1] - dp[n - 2]) * (10 - (n - 1))

dp[n-1] : 前面 n-1 位 unique 的数目

 (dp[n - 1] - dp[n - 2]) * (10 - (n - 1)) : 第n 位相对于 n-1 位上的增加的 unique 数量

dp[n-1]-dp[n-2]：指的是第n-1位较n-2 位多出来的unique数字。以n=3为例，n=2已经计算了0-99之间不重复的数字，需要求的是100-999之间unique的数字，只能用10-99之间unique的数去组成三位数，而不能使用0-9之间unique的数，因为一位数加一位数也组成不了3位数。而10-99之间unique的数等于dp[2]-dp[1]。

10 - （n - 1）:10个数字中还剩下未选的数字

## Code

```kotlin
class Solution {
    fun countNumbersWithUniqueDigits(n: Int): Int {
        if (n == 0) {
            return 1
        }
        val dp = IntArray(n + 1)
        dp[0] = 1
        dp[1] = 10
        for (i in 2..n) {
            dp[i] = dp[i - 1] + (dp[i - 1] - dp[i - 2]) * (10 - (i - 1))
        }
        return dp[n]
    }
}
```