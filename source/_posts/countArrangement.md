title:   Beautiful Arrangement
date: 2020-3-24
tags: [Leetcode,回溯]
categories: algorithm

description: 排列题目
---

## Problem description

https://leetcode-cn.com/problems/beautiful-arrangement/

## Solution

和之前的[全排列ii](https://nightxlt.github.io/2020/03/02/permuteUnique/) 类似，在全排列的基础上加了一个判断条件进行剪枝，不过自己在这里面考虑是有误的，自己当时想的是交换后两边都必须满足约束整除条件方可放置，但这样致使了漏解的情况，在 N=6时，Output = 34，正确答案是 36.  其实并不需要交换后的两个位置都满足条件，只需要交换后的头部元素满足条件即可，剩余元素如果不满足可以放到后面去进行逐步交换调整。故只需要判断交换过来的 nums[i]满足整除条件即可。

## Code

```java
class Solution {
    
    var count = 0

    fun countArrangement(N: Int): Int {
        if (N <= 0) {
            return -1
        }
        permute(IntArray(N) { it + 1 })
        return count
    }

    fun permute(nums: IntArray): List<List<Int>> {
        val res = mutableListOf<List<Int>>()
        DFS(nums, 0, res)
        return res
    }

    private fun swap(nums: IntArray, m: Int, n: Int) {
        if (m == n) {//自己和自己交换个鬼呀
            return
        }
        nums[m] = nums[m] xor nums[n]
        nums[n] = nums[m] xor nums[n]
        nums[m] = nums[m] xor nums[n]
    }

    fun DFS(nums: IntArray, start: Int, lists: MutableList<List<Int>>) {
        if (start == nums.size) {
            count++
            return
        }
        for (i in start until nums.size) {
            if (isSwap(nums[start], i) && isSwap(nums[i], start)) {
                swap(nums, start, i)
                DFS(nums, start + 1, lists)
                swap(nums, start, i)
            }
        }
    }

    fun isSwap(num: Int, index: Int): Boolean {
        return num % (index + 1) == 0 || (index + 1) % num == 0
    }
}
```