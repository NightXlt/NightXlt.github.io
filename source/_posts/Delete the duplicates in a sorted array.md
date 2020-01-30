title: Delete the duplicates in a sorted array
date: 2019-1-17
tags: [Leetcode,字节跳动]
categories: algorithm
description: O(1) extra memory
---
## Problem description
  ### Given a sorted array nums, remove the duplicates in-place such that each element appear only once and return the new length.Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.
 ## Examples
``` java
Example 1:
Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.

It doesn't matter what you leave beyond the returned length.
```
```java
Example 2:
Given nums = [0,0,1,1,1,2,2,3,3,4],

Your function should return length = 5, with the first five elements of nums being modified to 0, 1, 2, 3, and 4 respectively.

It doesn't matter what values are set beyond the returned length.
```

## Solution
　　(。﹏。*)意识到要紧抓排序这个特征，但一味想着找到不同的两个，然后整个数组往前移然后循环套循环立即推懵B状态。参考了别人的思路。牛皮。竟然用到了快慢指针。是在下庸俗了，总以为是用在链表找中间节点或环形链表。用快慢指针找到最近两个不相等的元素，然后直接赋值。
## Code

```java
 class Solution {
      public int removeDuplicates(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        int slow = 0, count = 1;
        for (int fast = 0; fast < nums.length; fast++) {
            if (nums[slow] != nums[fast]) {
                slow++;
                count++;
                nums[slow] = nums[fast];
            }
        }
        return count;
    }

}
```