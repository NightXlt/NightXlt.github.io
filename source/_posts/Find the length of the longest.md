title: Find the length of the longest consecutive elements sequence.
date: 2019-1-1
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given an unsorted array of integers, find the length of the longest consecutive elements sequence.Your algorithm should run in O(n) complexity.
 ## Examples
``` java
Example 1:
Input: [100, 4, 200, 1, 3, 2]
Output: 4
Explanation: The longest consecutive elements sequence is [1, 2, 3, 4]. Therefore its length is 4.
```
## Solution
　　一开始想通过计数排序，再统计最大连续长度。忽略题目未说是正数呀。遇到负数就炸了。用HashMap来存当前值处的连续区间长度。当插入一个数时，取出当前值前后的连续区间长度left，right。当前的区间长度 = left + right. + 1。.复杂问题优先考虑分治，拆成一个个小问题。假设左边已经是排好，右边已经排好。插入中间值，仅需将左侧长度+右侧长度+1即为整块区间长度。

## Code

```java
 class Solution {
    
public int longestConsecutive(int[] nums) {
        Map<Integer, Integer> map = new HashMap<>();
        int low, high, sum;
        int maxLength = 0;
        for (int i = 0; i < nums.length; i++) {
            if (map.containsKey(nums[i])) {
                continue;
            }
            low = map.containsKey(nums[i] - 1) ? map.get(nums[i] - 1) : 0;
            high = map.containsKey(nums[i] + 1) ? map.get(nums[i] + 1) : 0;//get both sides' consecutive information.
            sum = low + high + 1;
            maxLength = Math.max(maxLength, sum);
            map.put(nums[i], sum);
            map.put(nums[i] - low, sum);
            map.put(nums[i] + high, sum);//merge
        }
        return maxLength;
    }
}
```