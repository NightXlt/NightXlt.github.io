title:   Word Break
date: 2020-4-15
tags: [Leetcode,动态规划]
categories: algorithm

description: 单词拆分
---

## Problem description

https://leetcode-cn.com/problems/word-break/

## Solution

动规方程
$$
dp[i] = dp[j]\  \& \ wordDict.contains(j,i)\ \ \ 0\le j \lt i
$$
这里的 dp[i] 表示的是0..i - 1是否可以拆分为 dict 中的单词。对于s 中的每个字符都遍历一遍它前面的字符。看是否满足动规方程。能够满足就 break，表示我们找到了前 i 个字符的拆分方法，没必要再遍历下去了。

## Code

```java
class Solution {
    fun wordBreak(s: String, wordDict: List<String>): Boolean {
        val dp = Array(s.length + 1) { false }
        dp[0] = true
        for (i in 1..s.length)
            for (j in 0 until i)
                if (dp[j] && wordDict.contains(s.substring(j, i))) {
                    dp[i] = true
                    break
                }
        return dp[s.length]
    }
}
```