title:   Word BreakII
date: 2020-4-18
tags: [Leetcode,回溯]
categories: algorithm

description: 单词拆分II
---

## Problem description

https://leetcode-cn.com/problems/word-break-ii/

## Solution

类似上道题[word break](https://nightxlt.github.io/2020/04/15/wordBreaks/) 用普通回溯会超时，不过没有再使用 dp，而是使用记忆化回溯。用一个 Map 记录下每个 start 深搜后遍历得到的结果进行剪枝。之后再次遍历到 start 时，直接从 map 中取即可，有点缓存的意思。这道题如果用 dp 做，思路和之前类似，就变了一点 dp 中存的是 list，而且 dp 在面对长字符串是会出现超时的情况，因为dp会进行存储大量的中间结果如case: aaa...b...aaa，导致内存溢出。

## Code

```java
class Solution {
fun wordBreak(s: String, wordDict: List<String>): List<String> {
    if (s.isEmpty()) return emptyList()
    return dfs(s, wordDict.toSet(), 0)
}

val map = HashMap<Int, List<String>>()

fun dfs(s: String, wordDict: Set<String>, start: Int): List<String> {
    if (start == s.length) return emptyList()
    if (map.containsKey(start)) return map[start]!!
    val res = mutableListOf<String>()
    for (i in start until s.length) {
        val subString = s.substring(start, i + 1)
        if (!wordDict.contains(subString)) continue
        if (i + 1 == s.length) {
            res.add(subString) // 找到最后一段时直接跳出循环
            continue
        }
        val list = dfs(s, wordDict, i + 1)
        for (string in list) res.add("$subString $string")
    }
    map[start] = res
    return res
	}
}
```