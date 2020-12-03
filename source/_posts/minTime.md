title:  Minimum Time to Collect All Apples in a Tree
date: 2020-5-10
tags: [Leetcode,周赛]
categories: algorithm

description: 收集树上所有苹果的最少时间
---

## Problem description

https://leetcode-cn.com/problems/minimum-time-to-collect-all-apples-in-a-tree/

## Solution

从后往前同步子节点状态给父节点。再遍历一遍子节点的状态得到路径数。

## Code

```kotl
class Solution {
    fun minTime(n: Int, edges: Array<IntArray>, hasApple: List<Boolean>): Int {
        val hasApple = hasApple as MutableList
        for (i in edges.size - 1 downTo 0) {
            if (hasApple[edges[i][1]]) {
                hasApple[edges[i][0]] = true
            }
        }
        var res = 0
        for (i in 0 until edges.size) {
            if (hasApple[edges[i][1]]) {
                res += 2
            }
        }
        return res
    }
}
```