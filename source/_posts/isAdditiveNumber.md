title:    Additive Number
date: 2020-3-21
tags: [Leetcode,深搜]
categories: algorithm

description: 累加数
---

## Problem description

https://leetcode-cn.com/problems/additive-number/

## Solution

异常烦躁的看到这道题，大致思路是对的通过深搜 start..length, 判断前两项的和是否等于第三项，等于的话将 sum  + num3 作为前两项的和继续深搜，但是自己在细节处理的不够到位磕磕碰碰一直没有 ac。

比如遍历时数组首两项时，是没有前两项和的，需要直接返回前两项的元素。

从 num 从取子串出现 01，02这样的子串是非法的，而 0 是合法的。

其中含有的测试 case 会出现超过 Long 的大数情况，需要自己实现大数加法

## Code

```kotlin
class Solution {
    fun isAdditiveNumber(num: String): Boolean {
        if (num.length < 3) {
            return false
        }
        return dfs(num, "0", "0", 0, 0)
    }

    fun dfs(num: String, preSum: String, preNum: String, start: Int, k: Int): Boolean {
        if (k > 2 && start >= num.length) {
            return true
        }
        var len = 1
        while (len + start <= num.length) {
            val sum = isSum(num, start, start + len, preSum, k)
            if (sum != "-1") {
                if (dfs(num, add(sum, preNum), sum, start + len, k + 1)) return true
            }
            len++
        }
        return false
    }

    private fun isSum(num: String, start: Int, end: Int, preSum: String, k: Int): String {
        if (num[start] == '0' && start < end - 1) return "-1"
        val sum = num.substring(start, end)
        if (k < 2) return sum
        return if (preSum == sum)
            sum
        else
            "-1"
    }

    //两个字符串相加
    fun add(str1: String, str2: String): String {
        var str2 = str2
        if (str1.length < str2.length)
            return add(str2, str1)

        val size1 = str1.length
        val size2 = str2.length
        val sb = StringBuilder()
        for (i in 0 until size1 - size2) {
            sb.append('0')
        }
        str2 = sb.toString() + str2

        val result = StringBuilder(str1)
        var sgn = 0
        for (i in size1 - 1 downTo 0) {
            val num = str1[i] - '0' + str2[i].toInt() - '0'.toInt() + sgn
            result.setCharAt(i, (num % 10 + '0'.toInt()).toChar())
            sgn = num / 10
        }
        return if (sgn == 1) "1$result" else String(result)
    }

}
```