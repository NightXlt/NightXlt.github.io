title: Find all possible permutations
date: 2019-1-26				
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a collection of distinct integers, return all possible permutations.The order of list doesn't matter.
 ## Examples
``` java
Example 1:
Input: [1,2,3]
Output:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```
## Solution
　　一种解法两种出发角度：
1. 第一次见到的交换+回溯。遍历数组，每遇到一个元素nums[i]，将其与nums[start]交换，再对 i 之后的元素递归执行深搜。找到长度为length - 1的集合后加入结果集。再将 nums[i] 与 nums[start] 交换回来，否则会出现重复集合。
2. 分治思想将该字符串分为两部分：第一部分为它的第一个字符，第二部分为剩余所有字符。假设第二部分已找到全排列，加上第一部分。就得到以该字符为头的全排列。将剩余所有字符与首字符交换。即可得到所有字符的全排列.如下图所示，字符串划分为两部分：红色首字符和蓝色剩余字符。

![enter description here](/images/all_permutations.png)
	遍历所有字符
（1）将当前字符i与第一个字符交换
（2）再求出i之后剩余字符的全排列(递归)
（3）再将当前字符与首字符交换回来

## Code

```java
 class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> lists = new ArrayList<>();
        DFS(nums, 0, lists);
        return lists;
    }
    private void swap(int[] nums, int m, int n) {
        int temp = nums[m];
        nums[m] = nums[n];
        nums[n] = temp;
    }
    public void DFS(int[] nums, int start, List<List<Integer>> lists) {
        if (start == nums.length) {
            List<Integer> list = new ArrayList<>();
            for (int num : nums) {
                list.add(num);
            }
            lists.add(list);
            return;
        }
        for (int i = start; i < nums.length; i++) {
            swap(nums, start, i);
            DFS(nums, start + 1, lists);
            swap(nums, start, i);
        }
    }
}
```