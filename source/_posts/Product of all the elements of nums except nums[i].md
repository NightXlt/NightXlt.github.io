title: Product of all the elements of nums except nums[i]
date: 2019-1-19
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given an array nums of n integers where n > 1,  return an array output such that output[i] is equal to the product of all the elements of nums except nums[i].Please solve it without division and in O(n).
 ## Examples
``` java
Example 1:
Input:  [1,2,3,4]
Output: [24,12,8,6]
```

## Solution
　　一开始用除法试了下，发现有含0的样例，然后用了O(n^2)过了。自己思考过正向遍历记录每一个元素左侧的元素累积。但是这样原数组也变了。无法进行逆向遍历。只能采取O(n)的空间复杂度。新建一个数组先记录正向乘积。再逆向遍历一次乘以每个元素右侧的累积

## Code

```java
     public int[] productExceptSelf(int[] nums) {
        if (nums == null || nums.length == 0) {
            return nums;
        }
        int result[] = new int[nums.length];
        result[0] = 1;
        for (int i = 1; i < nums.length; i++) {
            result[i] = result[i - 1] * nums[i - 1];
        }
        int mul = 1;
        for (int i = nums.length - 1; i >= 0; i--) {
            result[i] *= mul;
            mul *= nums[i];
        }
        return result;
    }

```