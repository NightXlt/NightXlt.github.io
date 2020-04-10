title:  Wildcard Matching
date: 2020-4-9
tags: [Leetcode,动态规划,回溯]
categories: algorithm

description: 通配符匹配
---

## Problem description

https://leetcode-cn.com/problems/wildcard-matching/

## Solution

### 动态规划

和[正则匹配](https://nightxlt.github.io/2020/04/04/REMatch/) 那道题类似，不过本题用了从前至后遍历.

dp[i]\[j] 表示 s[0..i] 与 p[0..j]相匹配

其中 dp[i]\[0](i > 0) ：p 为空串，s不为空串，那么 dp[i]\[0]都为 false； dp[0]\[j](j > 0) 表示： s 为空串，p 不为空串，只有当 p 全为\*时, dp[0]\[j] 才会为 true。

当遇到字符为 \* 时，dp[i]\[j] =  dp[i - 1]\[j] || dp[i]\[j - 1], 前者表示 \* 重复一次，后者表示匹配空串。

此外when 的语法糖进行解糖后就是 switch case。



### 回溯法

回溯法是参考官方题解，使用两个指针 sInde和 pIndex

```kotlin
While sIndex < s.length
 if pIndex < p.length && p[pIndex] == *
		首先看匹配空串的情况, pIndex++,记录下这时 pIndex 和 sIndex便于之后匹配失败用来回溯
 if 不匹配
		if 之前的字符串没有出现过 * ，返回 false
		if 之前字符串出现过 * ，进行回溯。 pIndex = pTemp + 1, sIndex = sTemp + 1, 因为 * 可能匹配多个字符，所以 sTemp = sIndex,便于之后再次匹配失败后的回溯

if sIndex == s.length && pIndex..P.length == *，返回 true.
```

​	

## Code

```kotl
dp
class Solution {
    fun isMatch(s: String, p: String): Boolean {
        if (p.isEmpty()) return s.isEmpty()
        val sLen = s.length
        val pLen = p.length
        val dp = Array(sLen + 1) { BooleanArray(pLen + 1) }
        dp[0][0] = true
        for (j in 1..pLen)
            dp[0][j] = (p[j - 1] == '*') && dp[0][j - 1]
        for (i in 1..sLen)
            for (j in 1..pLen) {
                dp[i][j] = when (p[j - 1]) {
                    '*' -> dp[i - 1][j] || dp[i][j - 1]
                    '?' -> dp[i - 1][j - 1]
                    else -> dp[i - 1][j - 1] && s[i - 1] == p[j - 1]
                }
            }
        return dp[sLen][pLen]
    }
}
```



```java
回溯
class Solution {
    fun isMatch(s: String, p: String): Boolean {
        val sLen = s.length
        val pLen = p.length
        var sIdx = 0
        var pIdx = 0
        var starIdx = -1
        var sTmpIdx = -1

        while (sIdx < sLen) {
            // If the pattern caracter = string character
            // or pattern character = '?'
            if (pIdx < pLen && (p[pIdx] == '?' || p[pIdx] == s[sIdx])) {
                ++sIdx
                ++pIdx
            } else if (pIdx < pLen && p[pIdx] == '*') {
                // Check the situation
                // when '*' matches no characters
                starIdx = pIdx
                sTmpIdx = sIdx
                ++pIdx
            } else if (starIdx == -1) {
                return false
            } else {
                // Backtrack: check the situation
                // when '*' matches one more character
                pIdx = starIdx + 1
                sIdx = sTmpIdx + 1
                sTmpIdx = sIdx
            }// If pattern character != string character
            // or pattern is used up
            // and there was '*' character in pattern before
            // If pattern character != string character
            // or pattern is used up
            // and there was no '*' character in pattern
            // If pattern character = '*'
        }

        // String is used up, and the remaining characters in the pattern should all be '*' characters
        for (i in pIdx until pLen)
            if (p[i] != '*') return false
        return true
    }

}
```