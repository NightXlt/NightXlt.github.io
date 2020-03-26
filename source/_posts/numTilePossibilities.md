title:   Letter Tile Possibilities
date: 2020-3-26
tags: [Leetcode,回溯]
categories: algorithm

description: 活字印刷
---

## Problem description

https://leetcode-cn.com/problems/letter-tile-possibilities/

## Solution

画出递归树

![递归树](/images/traversalTree.png)

括号中的数字表示字母的使用次数，我们使用一个 count 数组记录字母的出现次数，每次遍历 count 数组，找到一个出现次数大于零的字母，使 count[i]减一表示找到访问过该分支字母，然后继续深搜下去，当回溯回来时，count[i]++恢复状态。期间使用 res 来统计集合数目。

## Code

```java
class Solution {
    fun numTilePossibilities(tiles: String): Int {
        val count = IntArray(26) { 0 }
        tiles.forEach {
            count[it - 'A']++
        }
        fun backTrack(): Int {
            var res = 0
            for (i in count.indices) {
                if (count[i] <= 0) {
                    continue
                }
                res += 1
                count[i]--
                res += backTrack()
                count[i]++
            }
            return res
        }
        return backTrack()
    }
}
```