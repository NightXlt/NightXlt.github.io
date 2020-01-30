title: The kth largest element
date: 2019-1-1
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Find the kth largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.You may assume k is always valid, 1 ≤ k ≤ array's length.
 ## Examples
``` java
Example 1:
Input: [3,2,1,5,6,4] and k = 2
Output: 5
```
```java
Example 2:
Input: [3,2,3,1,2,4,5,5,6] and k = 4
Output: 4
```
## Solution
　　思路是利用快排找到第n.length - k个枢轴即可。疑惑的是，提交上去AC了但有些耗时（32组样例跑了40ms）。复杂度为O(n)。看了4msAC的代码。真的是太简单粗暴啦。直接调用库函数排序，再取第n.length - k个元素。我对这个算法的时间复杂度产生了疑问。考虑过排序再取但太过耗时。快排时间复杂度是O（nlogn）。所以JDK用的并非快排。源码中采取的是DualPivotQuicksort方法。具体分析放在下片博文里[dualpivotsort](https://nightxlt.github.io/2019/01/01/DualPivotSort/)。

## Code

```java
     public int partition(int[] nums, int low, int high) {
        int pivot = nums[low];
        while (low < high) {
            while (low < high && nums[high] >= pivot) {
                high--;
            }
            nums[low] = nums[high];
            while (low < high && nums[low] <= pivot) {
                low++;
            }
            nums[high] = nums[low];
        }
        nums[low] = pivot;
        return low;
    }

    int quickSort(int[] nums, int k, int low, int high) {
        int pos = nums.length - k;
        int pivot = partition(nums, low, high);
        if (pivot == pos) {
            return nums[pivot];
        } else if (pivot > pos) {
            return quickSort(nums, k, low, pivot - 1);
        } else {
            return quickSort(nums, k, pivot + 1, high);
        }
    }

    public int findKthLargest(int[] nums, int k) {
        if (nums.length == 1 && k == 1) {
            return nums[0];
        }
        return quickSort(nums, k, 0, nums.length - 1);
    }

```