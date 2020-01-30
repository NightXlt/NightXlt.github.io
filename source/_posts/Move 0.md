title: Move 0 
date: 2019-2-2
tags: [Leetcode,腾讯]
categories: algorithm
description: 奇偶数组排序类似问题
---
## Problem description
  ### Given an array nums, write a function to move all 0's to the end of it while maintaining the relative order of the non-zero elements.You must do this in-place without making a copy of the array.Minimize the total number of operations.
 ## Examples
``` java
Example 1:
Input: [0,1,0,3,12]
Output: [1,3,12,0,0]
```
## Solution
　　总觉得在哪见过这道题的解法思想。但自己解出来的却不尽人意。起初是求出每个非元素i前有count个为0的元素。交换nums[i] 与 nums[i - count]两个元素。可以维护两个索引，一个是新数组维护非零元素索引，一个是旧元素中非零元素索引。遍历旧数组，遇到非零元素就添加到新数组中。最后令新数组的末尾全为0.其实剑指Offer中的奇偶数组排序有异曲同工之妙。不过那道题是从两端逼近。一个索引是奇数，另一个是偶数索引。

## Code
```java
 class Solution {
public void moveZeroes(int[] nums) {
        if (nums == null || nums.length < 2) {
            return;
        }
        int nonzeroIndex = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != 0) {
                nums[nonzeroIndex++] = nums[i];
            }
        }
        int zeroIndex = nonzeroIndex;
        for (; zeroIndex < nums.length; zeroIndex++) {
            nums[zeroIndex] = 0;
        }
    }
}
```
```java
class Solution {
    public int[] sortArrayByParity(int[] A) {
        int oddIndex = 0, evenIndex = A.length - 1;
        while (oddIndex < evenIndex) {
            while (oddIndex < A.length - 1 && (A[oddIndex] & 0x1) == 0) {
                oddIndex++;
            }
            while (evenIndex > 0 && (A[evenIndex] & 0x1) != 0) {
                evenIndex--;
            }
            if (oddIndex < evenIndex) {
                swap(A, oddIndex, evenIndex);
            }
        }
        return A;
    }
    private void swap(int[] data, int index, int end) {
        int t = data[index];
        data[index] = data[end];
        data[end] = t;
    }
}
```