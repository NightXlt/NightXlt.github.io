title:   Pyramid Transition Matrix
date: 2020-2-22
tags: [Leetcode,递归]
categories: algorithm

description: 递归
---

## Problem description

  ### https://leetcode-cn.com/problems/pyramid-transition-matrix/


## Solution

既然是先给到的是底层，就先从底层开始搭建。每次我们从底层抽取两个字符选取他们在 allowed 中的对应字符进行搭建上一层，搭建到什么时候为止呢。搭建至上面一层的个数为下层个数减一时开始下一轮递归，以上层字符串作为当前串，置空上层串（新一轮的搭建过程），开始递归，一直`递归`搭建直至顶层个数为 1，而第二层个数为 2 时终止。

怎么记忆 allowed 中的允许字符呢？用一个 map <String, HashSet<Char>>进行记忆，因为一个 String 可能有多个匹配的上方字符，为了防止重复选取 HashSet 进行筛选。

如
allowed = ["XXX", "XXY", "XYX", "XYY", "YXZ"]
hashmap = {'XX': ['X', 'Y'], 'XY': ['X', 'Y'], 'YX': ['Z']}

因为在 HashSet 中存在多个可能不同的上方匹配字符，我们需要遍历所有可能，只要存在一种递归可能搭建成金字塔立即返回为 true.

时间复杂度： O(n^2)

空间复杂度： 未知

## Code

```java
class Solution {
    fun pyramidTransition(bottom: String, allowed: List<String>): Boolean {
        val map = HashMap<String, HashSet<Char>>()
        allowed.forEach {
            if (map[it.substring(0, 2)] == null) {
                map[it.substring(0, 2)] = HashSet()
            }
            map[it.substring(0, 2)]?.add(it[2])//建立底层字符和基层字符的映射关系
        }
        return recursion(bottom, "", map)
    }
//cur:当前字符， above: 上方字符 map：存储底层字符和基层字符的映射关系 
    private fun recursion(cur: String, above: String, map: java.util.HashMap<String, java.util.HashSet<Char>>): Boolean {
        if (cur.length == 2 && above.length == 1) {//搭建完毕
            return true
        }
        if (cur.length == above.length + 1) { //上层搭建完毕，开启新一轮搭建
            return recursion(above, "", map)
        }
        val pos = above.length
        val remainString = cur.substring(pos, pos + 2)
        if (map.containsKey(remainString)) {
            map[remainString]?.forEach {
                if (recursion(cur, above + it, map)) {
                    return true
                }
            }
        }
        return false
    }
}
```