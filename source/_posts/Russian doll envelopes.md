title:  Russian doll envelopes
date: 2019-1-10
tags: [Leetcode,字节跳动]
categories: algorithm
description: Find the maximum number of envelopes can you Russian doll? (put one inside other)
---
## Problem description
  ### You have a number of envelopes with widths and heights given as a pair of integers (w, h). One envelope can fit into another if and only if both the width and height of one envelope is greater than the width and height of the other envelope.

### What is the maximum number of envelopes can you Russian doll? (put one inside other)

### Note:Rotation is not allowed.

 ## Examples
``` java
Example 1:
Input: [[5,4],[6,4],[6,7],[2,3]]
Output: 3 
Explanation: The maximum number of envelopes you can Russian doll is 3 ([2,3] => [5,4] => [6,7]).
```
## Solution
　　 这是一个二维LIS（Longest Increasing Subsequence）问题，可以通过先以宽度进行递增排序从而降维处理，即宽度有序仅需处理长度即可。再求长度最长递增子序列，注意的是当宽度相同时，要取长度最小的序列。有一点贪心的思想。长度越小，可放入的信封相应的也就越多。   
　　求递增子序列长度时不能直接双层遍历O(n^2)，可通过二分查找来查找降低为O(nlogn). 这里二分的思想有些奥妙。先是创建H数组存储"子序列的"最大递增子序列的末尾元素。通过二分查找H数组中元素（原数组中的最大递增子序列的最末元素）小于ai的长度最大的最大递增子序列；再令H[low] = ai.将长度为low的最大递增子序列的末尾元素置为ai。如果low 等于lenth的话，意味着最长递增序列的长度为 low + 1，所以length++。
## Code

```java
 class Solution {
    public int maxEnvelopes(int[][] envelopes) {
        Arrays.sort(envelopes, ((o1, o2) -> {
            if (o1[0] != o2[0]) {
                return o1[0] - o2[0];
            }
            return o2[1] - o1[1];//select lower height
        }));

        int len = 0;
        int[] h = new int[envelopes.length];
        int low, high ,m;
        for (int[] envelope : envelopes) {
            low = 0;
            high = len - 1;
            while (low <= high) {
                m = (low + high) / 2;
                if ((h[m] < envelope[1])) {
                    low = m + 1;
                } else {
                    high = m - 1;
                }
            }
            h[low] = envelope[1];
            if (low == len) {
                len++;
            }
        }
        return len;
    }
}
```