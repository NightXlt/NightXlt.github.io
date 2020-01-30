title: Implement strStr()
date: 2020-1-30
tags: [Leetcode]
categories: algorithm

description: comparator的一些事
---

## Problem description

Return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

links: https://leetcode-cn.com/problems/implement-strstr/

## Solution

参考：

https://www.cnblogs.com/ciyeer/p/9035072.html

https://www.cnblogs.com/ciyeer/p/9035072.html

Kmp 题目，复习了 Kmp 的相关知识。（想起了 DS 老师讲解这块时费力的身姿...）

以链接中的例子为例。

主串： ababcabcacbab

模式串： abcabac

Kmp 算法的核心是找到模式串中当第 j 个位置的字符不匹配时，应该从何处 next[j] 重新开始匹配. 关键是如何找到模式串的abcac next[j].

计算公式

![kmp 计算公式](/images/kmp.png)

公式最下方怎么理解呢？借助一张图

![kmp原理](/images/kmp_principle.png)

当主串中第i 个元素与模式串中的第 j 个元素不同时，这时应当将 j 滑到哪里呢？

首先当主串中第i 个元素与模式串中的第 j 个元素不同时，这意味着 i - j...i-1与1..j-1是相等的。

如果我们能找到一个 最大的K 值满足上图中的条件

即模式串中下标为 1...k-1的元素与j - k .. j-1元素相同，这就意味着下标 1...k-1的元素与 i - k.. i - 1相等。这两块相等就意味着我们可以让模式串滑到 k 处，从 k 这里开始匹配；即 next[j] = k；此外我们希望这个 K 值尽可能的大，因为这样剩余尚需匹配的模式串数目就少。

方便记忆，以匹配串“abcabac”为例求其的next数组。next 的下标从 1开始，j = 0。

```java
当 j = 0时'a'， 子串已经在最左侧了，无法再往左移动 j 了，应该移动 i元素。 令next[1] = 0作为标识

j = 1时'b'，当第二个元素不相等时，next[2] = 1表示应该与模式串中第一个元素进行比较

j = 2第三个字符 ‘c’ ：由于前一个字符 ‘b’ 的 next 值为 1 ，取 T[1] = ‘a’ 和 ‘b’ 相比较，不相等，变更 j = 2为 next[j] = 1 继续比较；由于 next[1] = 0，结束。 ‘c’ 对应的 next 值为1；（只要循环到 next[1] = 0 ,该字符的 next 值都为 1 ）

模式串T为：          “abcabac”
next数组(下标从1开始)：011

第四个字符 ’a‘ ：由于前一个字符 ‘c’ 的 next 值为 1 ，取 T[1] = ‘a’ 和 ‘c’ 相比较，不相等，继续；由于 next[1] = 0 ，结束。‘a’ 对应的 next 值为 1 ；

模式串T为：          “abcabac”
next数组(下标从1开始)：0111

第五个字符 ’b’ ：由于前一个字符 ‘a’ 的 next 值为 1 ，取 T[1] = ‘a’ 和 ‘a’ 相比较，相等，结束。 ‘b’ 对应的 next 值为：1(前一个字符 ‘a’ 的 next 值) + 1 = 2 ；

模式串T为：          “abcabac”
next数组(下标从1开始)：01112

第六个字符 ‘a’ ：由于前一个字符 ‘b’ 的 next 值为 2，取 T[2] = ‘b’ 和 ‘b’ 相比较，相等，所以结束。‘a’ 对应的 next 值为：2 (前一个字符 ‘b’ 的 next 值) + 1 = 3 ；

模式串T为：          “abcabac”
next数组(下标从1开始)：011123

第七个字符 ‘c’ ：由于前一个字符 ‘a’ 的 next 值为 3 ，取 T[3] = ‘c’ 和 ‘a’ 相比较，不相等，继续；由于 next[3] = 1 ，所以取 T[1] = ‘a’ 和 ‘a’ 比较，相等，结束。‘a’ 对应的 next 值为：1 ( next[3] 的值) + 1 = 2 ；

模式串T为：          “abcabac”
next数组(下标从1开始)：0111232
```



## Code

```kotlin
class Kmp {
    lateinit var next: IntArray
    fun strStr(haystack: String, needle: String): Int {
        if (haystack.length < needle.length) {
            return  -1
        }
        if (haystack.isEmpty() && haystack == needle) {
            return 0
        }
        initNext(needle)
        var i = 1
        var j = 1
        while (i <= haystack.length && j <= needle.length) {
            if (j == 0 || haystack[i - 1] == needle[j - 1]) {
                i++
                j++
            } else {
                j = next[j]
            }
        }
        if (needle.length in 1 until j) {
            return i - needle.length - 1
        }
        return -1
    }

    private fun initNext(needle: String) {
        if (needle.isBlank()) {
            next = IntArray(1){0}
            return
        }
        next = IntArray(needle.length + 1)
        next[0] = needle.length
        next[1] = 0
        var i = 1
        var j = 0 
        while (i < next[0]) {
            if (j == 0 || needle[i - 1] == needle[j - 1]) {
                next[++i] = ++j
            } else {
                j = next[j] //不相等时匹配串滑到对应位置 next[j],j - 1处元素继续和 i-1处元素进行比较
            }
        }
    }
}

fun main() {
    print(Kmp().strStr("aaasdsdfsdfsdf", "abcabac"))
}
```





