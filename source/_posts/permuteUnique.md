title:   Permutations II
date: 2020-3-2
tags: [Leetcode,回溯]
categories: algorithm

description: 不重复的全排列
---

## Problem description

https://leetcode-cn.com/problems/permutations-ii/

## Solution

和之前的[全排列](https://nightxlt.github.io/2019/01/26/Find all possible permutations/)不同的是加了不允许重复，这就需要在进行交换前进行一步判断，如果从 start 到字符 i 前就已经出现了i就不需要进行交互和深搜了，因为i 已经出现过且已经进行了深搜，再深搜下去只会重复。通过异或交换数字时，一定要保证是两个数字，只有一个数字和自己进行交换的话会将该数字变为 0 的。

## Code

```java
class Solution {
    fun permuteUnique(nums: IntArray): List<List<Int>> {
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
            val list = ArrayList<Int>()
            for (num in nums) {
                list.add(num)
            }
            lists.add(list)
            return
        }
        for (i in start until nums.size) {
            if (isSwap(nums, start, i)) {//判断 下标从start..i - 1的数字是否有和 nums[i]重复的，重复则不进行交换
                swap(nums, start, i)
                DFS(nums, start + 1, lists)
                swap(nums, start, i)
            }
        }
    }

    fun isSwap(nums: IntArray, start: Int, end: Int): Boolean {
        for (i in start until end) {
            if (nums[i] == nums[end]) {
                return false
            }
        }
        return true
    }
}
```