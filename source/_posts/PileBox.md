title:  Pile Box LCCI
date: 2020-4-11
tags: [Leetcode,动态规划]
categories: 程序员面试金典

description: 堆箱子
---

## Problem description

https://leetcode-cn.com/problems/pile-box-lcci/

## Solution

这是一个三维的 LIS，那可以对其进行排序后降维吧。这个采用 贪心 + 二分的方法是行不通的。老老实实用动态规划解决吧。

用动规就需要找动规方程，在数组 nums最大递增子序列中 dp[i]表示以 nums[i]结尾的最长上升序列数目。那么动规方程为
$$
dp[i] = max(dp[j]) + 1\\ 其中 0 \le j \lt i,\ 且 nums[j] < nums[i]
$$
那么 dp 数组中的最大值即为最大递增子序列的的长度， 即LIS(length) =  max(dp[i])。

讲人话是，我们对于每一个 nums[i], 遍历它前面的元素（0..i - 1），判断其元素值是否比它小，小的话，表明该元素可能是其递增序列中的一员。再对这些小于它的元素取 max 后 + 1得到结果。

在本题中 dp[i] 表示的是] 以 nums[i]结尾的最长递增子序列的累加和，便于计算箱子的高度。

时间复杂度： $O(n^2)$

空间复杂度： $O(n)$

## Code

```java
class Solution {
	fun pileBox(box: Array<IntArray>): Int {
    if (box.isEmpty()) return 0
    Arrays.sort(box) { a, b -> a[2] - b[2] } // 排序降维
    val dp = Array(box.size) { 0 }
    dp[0] = box[0][2]
    var res = 0
    for (i in 0 until box.size) {
        var max = box[i][2]
        for (j in 0 until i)
            if (box[i][0] > box[j][0] && box[i][1] > box[j][1] && box[i][2] > box[j][2])
                max = kotlin.math.max(max, dp[j] + box[i][2])
        dp[i] = max
        res = kotlin.math.max(max, res)
    }
    return res
	}
}
```