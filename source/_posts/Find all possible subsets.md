title: Find all possible subsets
date: 2019-1-26
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a set of distinct integers, nums, return all possible subsets (the power set).Note: The solution set must not contain duplicate subsets.
 ## Examples
``` java
Example 1:
Input: nums = [1,2,3]
Output:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```


## Solution
　　和上一道 Find all possible permutations 类似，先将空集添入集合。
  再遍历数组：
  1. 将nums[i]添入list
  2. 再以每个元素nums[i]为头深搜出 nums[i] 与 nums[i] 元素之后构成的子集
  3. 再从list中删除nums[i]元素。
## Code

```java
 class Solution {
   
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> lists = new ArrayList<>();
        List<Integer> list = new ArrayList<>();
        Arrays.sort(nums);
        DFS(nums, 0, lists, list);
        return lists;
    }

    public void DFS(int[] nums, int start, List<List<Integer>> lists, List<Integer> list) {
        List<Integer> integers = new ArrayList<>(list);
        lists.add(integers);
        for (int i = start; i < nums.length; i++) {
            integers.add(nums[i]);
            DFS(nums, i + 1, lists, integers);
            integers.remove(integers.size() - 1);
        }
    }
}
```