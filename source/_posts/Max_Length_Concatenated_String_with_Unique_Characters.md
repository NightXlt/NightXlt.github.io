title:  串联字符串的最大长度
date: 2020-2-10
tags: [Leetcode,DFS]
categories: algorithm

description: dfs + 位压缩
---

## Problem description

  ### https://leetcode-cn.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/

## Solution

参考：https://leetcode-cn.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/solution/jian-ji-de-chui-su-yi-dong-by-huwt/

对于的数组中每一个字符串A，

1. 如果当前字符串加上串 A 没有重复字符
   1. 有两种可能添加或者不添加，我们只需比较添加该数的结果以及未添加结果的长度的最大值即可得到最终结果。类似二叉树的 dfs，
2. 如果当前字符串加上串 A 有重复字符，应该直接跳过串 A

位压缩： 指的用若干位数字 N 记录数字，字符等的存在状态，如果 1 << x 位（x 为整数）& N 不为零，表示 x 处的字符存在.如 26 位表示字母是否存在。1 << 0 表示第一个字符 ‘a’.  如果 1 << 0 & N 不为零，表示有 a 这个字符。参考https://blog.csdn.net/Geecky/article/details/52211373

## Code

```java
import kotlin.math.max

class Solution {
       private var t: Int = 0
    fun maxLength(arr: List<String>): Int {
        return dfs(arr, 0, 0)
    }

    private fun dfs(arr: List<String>, childIndex: Int, m: Int): Int { //用来存储当前结果小写字母出现的次数，只用了 26 位记录
        if (childIndex == arr.size) { //dfs 到底了
            return 0
        }
        t = m //t 用来存储 m 加上 str 中小写字母的次数
        val str = arr[childIndex]
        if (isUnique(str)) { //如果结果加上 str 没有重复字符
            val addChild = str.length + dfs(arr, childIndex + 1, t) //添加 str 的长度，继续深搜
            val noAddChild = dfs(arr, childIndex + 1, m) //跳过 str 继续深搜
            return max(addChild, noAddChild) //比较二者长度
        }
        return dfs(arr, childIndex + 1, m) //结果加上 str 有重复，则结果为 跳过 str 继续深搜的长度
    }

    private fun isUnique(str: String): Boolean {
        val length = str.length
        for (i in 0 until length) {
            if (t and (1 shl str[i] - 'a') != 0) { //如果有重复字符，直接返回
                return false
            }
            t = t or (1 shl str[i] - 'a')
        }
        return true
    }
}
```