title:   Bitwise ORs of Subarrays
date: 2020-2-19
tags: [Leetcode,位运算]
categories: algorithm

description: 　
---

## Problem description

  ### https://leetcode-cn.com/problems/bitwise-ors-of-subarrays/


## Solution

A | B 可能会将 A中的一位或者多位 0 -> 1，不存在 1 -> 0的情况

A，B的最大为 10^9,即最多为 30位, 当 A| B时，至多有 30为从 0变为 1.

 A[j]： 表示 从数组下标从 j 处到i处的元素进行或操作结果。

 A[i] & A[j]：表示 A[i] 从 A[j]中取出 A[i]的标志位。如果结果等于 A[i], 则表明 A[j]中含有 A[i]所有的二进制 1,就省去了再去进行与操作的步骤。这就是剪枝，通过必要的判断舍去部分的耗时分支执行时间。[剪枝](https://blog.csdn.net/u010700335/article/details/44079069)

## Code

```java
class Solution {
    fun subarrayBitwiseORs(A: IntArray): Int {
        if (A.size <= 1) {
            return 1
        }
        val res = HashSet<Int>()
        for (i in 0 until A.size) {
            res.add(A[i]) //每个数都是一个子数组
            for (j in i - 1 downTo 0) {
                if (A[i] and A[j] == A[i]) break
                A[j] = A[j] or A[i]
                res.add(A[j])
            }
        }
        return res.size
    }
}
```