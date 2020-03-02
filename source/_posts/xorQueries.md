title:   XOR Queries of a Subarray
date: 2020-2-23
tags: [Leetcode,位运算]
categories: algorithm

description: 前 n 项和异或
---

## Problem description

  ### https://leetcode-cn.com/problems/xor-queries-of-a-subarray/


## Solution

参考https://leetcode-cn.com/problems/xor-queries-of-a-subarray/solution/zi-shu-zu-yi-huo-cha-xun-by-leetcode-solution/

令 pre 记录 arr 中的前 n - 1项异或和，

```
pre[i] = arr[0] ^ arr[1] ^ ... ^ arr[i - 1]
```

那么可以得到

```
pre[Li] ^ pre[Ri + 1] = (arr[0] ^ ... ^ arr[Li - 1]) ^ (arr[0] ^ ... ^ arr[Ri]) //Ri + 1将第 Ri 纳入到异或范围内
                      = (arr[0] ^ ... ^ arr[Li - 1]) ^ (arr[0] ^ ... ^ arr[Li - 1]) ^ (arr[Li] ^ ... ^ arr[Ri]) 
                      = arr[Li] ^ ... ^ arr[Ri]
```

确保角标不要有问题，pre 是从 1..arr.size, 而 query 内的取值是 0..arr.size-1. 

## Code

```java
class Solution {
    fun xorQueries(arr: IntArray, queries: Array<IntArray>): IntArray {
        val pre = IntArray(arr.size + 1)
        pre[0] = 0
        arr.forEachIndexed { index, num ->
            pre[index + 1] = pre[index] xor num
        }
        val res = IntArray(queries.size)
        queries.forEachIndexed { index, query ->
            res[index] = pre[query[0]]  xor pre[query[1] + 1]
        }
        return res
    }
}
```