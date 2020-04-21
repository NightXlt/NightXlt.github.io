title:   Stickers to Spell Word
date: 2020-4-21
tags: [Leetcode,动态规划]
categories: algorithm

description: 贴纸拼词
---

## Problem description

https://leetcode-cn.com/problems/stickers-to-spell-word/

## Solution

类似位运算中的状态记录，用位图Bitmap的方式记录 target 的每一位是否被选中。如 abc的子集ab对应数字i: 110。 target 的长度为 n，那么位图的长度 m为 1 << n。

dp[i] : 表示组成第 i 个集合所需的最小贴纸数目。那么最后我们只需返回 dp[m - 1] 表示 target 中所有字母均选中 即可。

将 dp 数组初始化为 Int.MAX （该集合无法拼出来）, dp[0]初始化为 0。

```kotlin
遍历所有集合 即 m 种

	如果当前子集 i 无法被拼出，其再加上 char 也不会被拼出。

	寻找 sticker 中的字符添加进当前的集合 i 中，看能否增添新的在 target 中出现但未出现在集合 i 中的字符。
		遍历所有的 sticker
		  记录下当前的集合 i 为 cur
		  遍历每个 sticker 中的字符c
				遍历 target 
					if target.contains(c) && !cur.contains(c)
					  cur.add(c)
					  break
		dp[cur] = min(dp[cur], dp[i + 1]) // 取 添加后的集合 cur 中的最小贴纸数和原始集合的最小贴纸数 + 1的最小值。 + 1是只遍历一张贴纸嘛，里面添加的字符都是一张贴纸内的。
				
return d[m - 1]
```

这里面再提下的就是 cur.contains(c)因为 cur 是数字嘛，怎么判断第 i 位是否是 1 呢？ 位运算。 `x & (1 >> j) == 0等于 0`

cur.add(c):  `cur |=  (1 << j)`

## Code

```java
import kotlin.math.min

class Solution {
    fun minStickers(stickers: Array<String>, target: String): Int {
        val n = target.length
        val m = 1 shl n
        val dp = IntArray(m) { Int.MAX_VALUE }
        dp[0] = 0
        for (i in 0 until m) {
            if (dp[i] == Int.MAX_VALUE) continue
            for (sticker in stickers) {
                var cur = i
                for (c in sticker) {
                    for (j in target.indices) {
                        if (target[j] == c && cur shr j and 1 == 0) {
                            cur = cur or (1 shl j)
                            break
                        }
                    }
                }
                dp[cur] = min(dp[cur], dp[i] + 1)
            }
        }
        return if (dp[m - 1] == Int.MAX_VALUE) -1 else dp[m - 1]
    }
}
```