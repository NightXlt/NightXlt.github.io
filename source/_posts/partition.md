title:    Palindrome Partitioning
date: 2020-3-8
tags: [Leetcode,回溯]
categories: algorithm

description: 分割字符串
---

## Problem description

https://leetcode-cn.com/problems/palindrome-partitioning/

## Solution

这道题的结构和全排列的类似，结果中的list中所有字符串按顺序拼接是完整的字符串 s, 保证结果集中包含所有元素，深搜时添加的是一个子串。为了判断这个子串是回文，一开始使用的是每个子串遍历进行比较，后面看了题解在判断是否为回文子串那里采用了最大回文子串的解法，可以使用动态规划进行空间换取时间，预先都遍历好相关数据，使用时直接用 dp 中的数据判断即可，不过当时自己在回文子串那里选用的是中心扩散法，当时自己权衡了下 中心扩散 的空间复杂度是O(1),而 dp 的是 O(n^2)。回忆下dp 求最大回文子串的方法。使用 dp[i] [j]表示 $s_i..s_j$是否是回文子串。当 i..j 长度大于 2时， s[i..i]是否为回文子串其实取决于 s[i + 1..j - 1]是否为回文子串 以及 s[i] == s[j]， 当 i..i 长度小于等于2时，则是 s[i] == s[j]。所以得到其状态转移方程如下：

$$ dp[i]\[j] = 
\left\{\begin{matrix}
dp[i+1]\[j - 1]\&\&s[i]\[j] & j - i > 2\\\
s[i] = s[j] & j - i <= 2
\end{matrix}\right.$$

时间复杂度： $O(2^n)$,因为每个数后面均可能发生截断成为子串

空间复杂度：$O(n^2)$

## Code

```java
import java.util.*

class Solution {
    fun partition(s: String): List<List<String>> {
        if (s.isEmpty()) {
            return emptyList()
        }
        val queue = ArrayDeque<String>()
        val res = mutableListOf<List<String>>()
        val dp = Array(s.length) { BooleanArray(s.length) }
        for (right in 0 until s.length) {
            for (left in 0..right) {
                if (s[right] == s[left] && (right - left <= 2 || dp[left + 1][right - 1])) {
                    dp[left][right] = true
                }
            }
        }
        fun dfs(start: Int) {
            if (start == s.length) {
                res.add(queue.toList())
                return
            }
            for (i in start until s.length) {
                if (!dp[start][i]) {
                    continue
                }
                queue.add(s.substring(start, i + 1))
                dfs(i + 1)
                queue.removeLast()
            }
        }
        dfs(0)
        return res
    }
}
```