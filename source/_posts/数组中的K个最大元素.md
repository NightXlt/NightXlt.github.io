title: 数组中的K个最大元素
date: 2019-3-22
tags: [Leetcode]
categories: algorithm
description: Heap Sort
---
## Problem description
  ### 给定一个数组nums,取出其中最大的K个数

## Solution
1. 直接Arrays.sort，取出最后K个数
2. 采取优先级队列(小顶堆实现)
3. 自己用大顶堆实现
## Code
```java
 //2
 class Solution {
    public int[] findKLargest(int[] nums, int k) {
        Queue<Integer> max = new PriorityQueue<Integer>();
        for (int n : nums) {
            if (max.size() < k) max.offer(n);//构造k个数的优先队列，最小
            else if (n > max.peek()) {
                max.poll();
                max.offer(n);
            }
        }
        int[] res = new int[k];
        for (int i = 0; i < k; i++) {
            res[i] = max.poll();
        }
        return res;
    }
}
```

```java
 class Solution {
     public int[] findKLargest(int[] nums, int k) {
        if (nums == null || nums.length <= 1) {
            return nums;
        }
        buildMaxHeap(nums);
        int[] res = new int[k];
        for (int i = nums.length - 1; i > nums.length - 1 - k; i--) {
            swap(nums, i, 0);
            res[nums.length - i - 1] = nums[i];
            maxHeap(nums, 0, i - 1);
        }
        return res;
    }

    private void buildMaxHeap(int[] nums) {
        for (int i = nums.length / 2; i >= 0; i--) {
            maxHeap(nums, i, nums.length);
        }
    }

    private void maxHeap(int[] nums, int index, int length) {

        int left = 2 * index + 1;

        if (left < length && left + 1 < length && nums[left] < nums[left + 1]) {
            left++;
        }
        if (left < length && nums[index] < nums[left]) {
            swap(nums, index, left);
            maxHeap(nums, left, length);
        }
    }

    private void swap(int[] nums, int i, int j) {
        int t = nums[i];
        nums[i] = nums[j];
        nums[j] = t;
    }
}
```
