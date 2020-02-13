title:  Maximum XOR of Two Numbers in an Array
date: 2020-2-9
tags: [Leetcode,贪心]
categories: algorithm

description: 　　
---

## Problem description

  ### https://leetcode-cn.com/problems/maximum-xor-of-two-numbers-in-an-array/


## Solution

参考题解

https://leetcode-cn.com/problems/maximum-xor-of-two-numbers-in-an-array/solution/li-yong-yi-huo-yun-suan-de-xing-zhi-tan-xin-suan-f/

1. 如果 a ^ b = c, a ^ c = b, b ^ c = a，用来检测最大前缀是否正确，假设前面 n - 1位已知，假设第 n 位为 1令其为 temp。那么这个 temp 是否真的是最大的前缀呢？通过前面的公式进行测试。如果 t 是正确的最大前缀，那么 temp  ^ set 内的前缀 = set 内的前缀。也就是说 temp ^ set 内的数字 的结果必然在 set 内存在。因为 a (set 内元素) ^ b（set内元素） = C（最大前缀），那最大前缀 C ^ a (set 内元素) = b (set 内元素)
2. 找到最大的值，尽可能找到二进制最高的位（贪心思想）
3. 通过位掩码 找到 前缀，mask & a = 从a 中取出 mask 中的标志位

## Code

```java
class Solution {
    fun findMaximumXOR(nums: IntArray): Int {
        var mask = 0
        var res = 0
        for (i in 31 downTo 0) { //贪心思想，优先找到二进制中最高的位，再依次定下依次低位
            mask = mask or (1 shl i)
            val set = HashSet<Int>()
            for (num in nums) {
                set.add(num and mask) //取出各数中对应 mask 的标志位
            }
            val t = res or (1 shl i)
            for (prefix in set) {
                if (set.contains(prefix xor t)) { //判断set内是否存在异或结果
                    res = t
                }
            }
        }
        return res
    }
}
```