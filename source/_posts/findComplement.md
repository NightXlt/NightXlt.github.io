title:   Number complement
date: 2020-2-15
tags: [Leetcode,位运算]
categories: algorithm

description: 　
---

## Problem description

  ### https://leetcode-cn.com/problems/number-complement/


## Solution

找补码，诶，看看异或得行不，5 ^ 7 = 2，为啥是 7 呢？因为 5：101,有三位需要取反嘛。既然是异或同样长度的 1，那先统计出 x 有多少位，再求出对应count全为 1的 y。异或即可。有没更好的办法呢？在求 x 的 count 时，可以求出对应长度的 全 1的 y 呢？是可以的，在 n 右移时，将 1不断左移，这样就可以得到 1 << count，再减个一就可以得到与 x 对应长度全一父数字了。

## Code

```java
class Solution1 {
    fun findComplement(num: Int): Int {
        var n = num
        var count = 0
        while (n != 0) {
            n = n shr 1
            count++
        }
        var x = 1
        var y = 1
        count--
        while (count != 0) {
            y = (x shl 1) + 1
            x = y
            count--
        }
        return num xor y
    }
}

class Solution2 {
     fun findComplement(num: Int): Int {
        var n = num
        var t = 1
        while (n != 0) {
            n = n ushr 1
            t = t shl 1
        }
        t--
        return num xor t
    }
}
```