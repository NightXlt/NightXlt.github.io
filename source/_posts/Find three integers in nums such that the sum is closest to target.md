title:  Find three integers in nums such that the sum is closest to target
date: 2019-1-16
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given an array nums of n integers and an integer target, find three integers in nums such that the sum is closest to target. Return the sum of the three integers. You may assume that each input would have exactly one solution.
 ## Examples
``` java
Example 1:
Given array nums = [-1, 2, 1, -4], and target = 1.
The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```

## Solution
　　知识迁移能力，一开始仅是在上道题[The sum of three elements is zero](https://nightxlt.github.io/%2F2018%2F12%2F30%2FThe%20sum%20of%20three%20elements%20is%20zero%2F)加了个判断，找到最小偏差。但是这样直接用是不对的.因为找到最小偏差时，不能同时移动j,k要判断才能移动索引。若sum > target,k--;sum < target,j++;sum == target ,return target;矛盾的特殊性啊~~~ 具体情况具体分析。不要死板。

## Code

```java
 public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        int j, k;
        int closetSum = 0;
        int minOffset = Integer.MAX_VALUE;
        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            j = i + 1;
            k = nums.length - 1;
            while (j < k) {
                int sum = nums[i] + nums[j] + nums[k];
                int offset = Math.abs(sum - target);
                if (sum > target) {
                    if (offset < minOffset) {
                        minOffset = offset;
                        closetSum = sum;
                        while (j < k && nums[k - 1] == nums[k]) {
                            k--;
                        }
                    }
                    k--;
                } else if (sum < target) {
                    if (offset < minOffset) {
                        minOffset = offset;
                        closetSum = sum;
                        while (j < k && nums[j + 1] == nums[j]) {
                            j++;
                        }
                    }
                    j++;
                } else {
                    return target;
                }
            }
        }
        return closetSum;
    }

```