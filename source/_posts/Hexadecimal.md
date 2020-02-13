title:  Convert a Number to Hexadecimal
date: 2020-2-1
tags: [Leetcode,位运算]
categories: algorithm

description: 　
---

## Problem description

  ### https://leetcode-cn.com/problems/convert-a-number-to-hexadecimal/


## Solution

转换十六进制有个技巧，四位一转，每次取出后四位abcd出来与 0xf 与下，得到的结果必然是abcd 代表的十进制数 i，再根据 i 从16 进制索引表中取出对应的 16 进制字符。小细节是，`每次需要将上次的结果放到后面去`。

## Code

```java
class Solution {
    fun toHex(num: Int): String {
        var n = num
        var res = ""
        val string = "0123456789abcdef"
        while (n != 0 && res.length < 8) {
            res = string[n and 0xf] + res
            n = n shr 4
        }
        return res
    }
}
```