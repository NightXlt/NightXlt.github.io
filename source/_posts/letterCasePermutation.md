title:   Letter Case Permutationt
date: 2020-2-21
tags: [Leetcode,回溯]
categories: algorithm

description: 字母大小写全排列
---

## Problem description

  ### https://leetcode-cn.com/problems/letter-case-permutation/


## Solution

参照： https://leetcode-cn.com/problems/letter-case-permutation/solution/hui-su-fa-c-by-di-wu-2/

基础的回溯思想题，也可以说是深搜，后加上了状态变化。每次深度搜索过后，变化当前字幕的大小写即可。深搜直至达到length 长度停止。

## Code

```java
class Solution {
    fun letterCasePermutation(S: String): List<String> {
        if (S.isEmpty()) {
            return listOf()
        }
        val list = mutableListOf<String>()
        dfs(0, StringBuilder(S), list)
        return list
    }

    fun dfs(i: Int, s: StringBuilder, list: MutableList<String>) {
        if (i != s.length) {
            if (s[i].isLowerCase()) {
                dfs(i + 1, s, list)
                s.setCharAt(i, s[i] - 32)
                dfs(i + 1, s, list)
            } else if (s[i].isUpperCase()) {
                dfs(i + 1, s, list)
                s.setCharAt(i, s[i] + 32)
                dfs(i + 1, s, list)
            } else {
                dfs(i + 1, s, list)
            }
        } else {
            list.add(s.toString())
        }
    }
}
```