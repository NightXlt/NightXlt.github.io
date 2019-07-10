title: The sum of three elements is zero
date: 2018-12-30
tags: [Leetcode,字节跳动]
categories: algorithm
description: Find all unique triplets in the array which gives the sum of zero.
---
## Problem Description
  ### Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
 ## Examples
``` java
Example 1:
Input:[-1, 0, 1, 2, -1, -4]
Output:[  [-1, 0, 1],[-1, -1, 2]]
```

## Solution
   一开始，思路是   for i=0 to n/2,    	 for j=n-1，再在内层循环内用二分查找判断三个元素是否等于0。出现重复元素，且有遗漏的情况。为了防止遗漏，for i = 0 to n-2;  j = i + 1;(防止重复)，k = n - 1.
   若 nums[i] + nums[j] + nums[k] > 0，说明k取大了，k--;
   若 nums[i] + nums[j] + nums[k] < 0，说明k取小了，k++;
   若nums[i] + nums[j] + nums[k] = 0，将三个数加入集合，再防止重复。降低时间复杂度。
   
   一个关于Java长久的坑。存在两个listA,listB集合。 listA.add(listB); listB.clear();这样刚刚添加到listA的集合也会为空。不能clear()。为防止数据叠加。每次new listB()即可。 Android曾遇到同样问题。
## Code

```java
     public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);

        List<List<Integer>> lists = new ArrayList<>();
        int j, k, sum;
        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            j = i + 1;
            k = nums.length - 1;
            while (j < k) {
                sum = nums[i] + nums[j] + nums[k];
                if (sum > 0) {
                    k--;

                } else if (sum < 0) {
                    j++;
                } else {
                    List<Integer> integers = new ArrayList<>();
                    integers.add(nums[i]);
                    integers.add(nums[j]);
                    integers.add(nums[k]);
                    lists.add(integers);
                    while (j < k && nums[j + 1] == nums[j]) {
                        j++;
                    }
                    while (j < k && nums[k - 1] == nums[k]) {
                        k--;
                    }
                    j++;
                    k--;
                }
            }

        }
        return lists;
    }

```