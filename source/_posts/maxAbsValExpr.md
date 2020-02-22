title:   Maximum of Absolute Value Expression
date: 2020-2-19
tags: [Leetcode,数学]
categories: algorithm

description: 曼哈顿距离　
---

## Problem description

  ### https://leetcode-cn.com/problems/maximum-of-absolute-value-expression/


## Solution

参考题解https://leetcode-cn.com/problems/maximum-of-absolute-value-expression/solution/miao-sha-ci-ti-jing-dian-man-ha-dun-ju-chi-suan-fa/

[曼哈顿距离](https://zh.wikipedia.org/wiki/曼哈頓距離)

可以看出这是一个三维的曼哈顿距离，将 arr1中的坐标当为 x,arr2 中的坐标当为 y，进行去绝对值后，共有八种可能。

当我们将第 i项的符号定下来时，那 j 项的符号也就随之定下来了。所以我们先求出在八种可能中每种可能的 i 项之和即 arr1[i] + arr2[i] + i,当然各项还需要乘以各自的符号求出最大值。

接下来，利用求出八种可能中的i项最大和 - 八种可能中 j 项的最大和。并求其最大值。

时间复杂度： O(2^N * N)

空间复杂度：O(2^N * N)

更优解法： https://leetcode-cn.com/problems/maximum-of-absolute-value-expression/solution/python-jie-fa-bao-li-shu-xue-by-jiayangwu/

## Code

```java
import kotlin.math.max

class Solution {
    val symbol = arrayOf(intArrayOf(1, 1, 1), intArrayOf(1, 1, -1), intArrayOf(1, -1, 1), intArrayOf(-1, 1, 1),
            intArrayOf(-1, -1, 1), intArrayOf(-1, 1, -1), intArrayOf(1, -1, -1), intArrayOf(-1, -1, -1))
    var maxI = IntArray(8){ Int.MIN_VALUE }
    fun maxAbsValExpr(arr1: IntArray, arr2: IntArray): Int {
        var res = Int.MIN_VALUE
        for (k in 0 until 8) {
            for (i in 0 until arr1.size) {
                maxI[k] = max(maxI[k], i * symbol[k][0] + arr1[i] * symbol[k][1] + arr2[i] * symbol[k][2])
            }
        }
        for (k in 0 until 8) {
            for (j in 0 until arr1.size) {
               res = max(res, maxI[k] - j * symbol[k][0] - arr1[j] * symbol[k][1] - arr2[j] * symbol[k][2])
            }
        }
        return res
    }
}
```