title:  Russian doll envelopes
date: 2019-1-10
tags: [Leetcode,字节跳动]
categories: algorithm

description: Find the maximum number of envelopes can you Russian doll? (put one inside other)
---
## Problem description
You have a number of envelopes with widths and heights given as a pair of integers (w, h). One envelope can fit into another if and only if both the width and height of one envelope is greater than the width and height of the other envelope.

What is the maximum number of envelopes can you Russian doll? (put one inside other)

Note:Rotation is not allowed.

 ## Examples
``` java
Example 1:
Input: [[5,4],[6,4],[6,7],[2,3]]
Output: 3 
Explanation: The maximum number of envelopes you can Russian doll is 3 ([2,3] => [5,4] => [6,7]).
```
## Solution
　　 这是一个二维LIS（Longest Increasing Subsequence）问题，可以通过先以宽度进行递增排序从而降维处理，即宽度有序仅需处理长度即可。再求长度最长递增子序列，注意的是当宽度相同时，要取长度最小的序列。有一点贪心的思想。长度越小，可放入的信封相应的也就越多。   
　　求递增子序列长度时不能直接双层遍历O(n^2)，可通过二分查找来查找降低为O(nlogn)。~~这里二分的思想有些奥妙。先是创建H数组存储"子序列的"最大递增子序列的末尾元素。通过二分查找H数组中元素（原数组中的最大递增子序列的最末元素）小于ai的长度最大的最大递增子序列；再令H[low] = ai.将长度为low的最大递增子序列的末尾元素置为ai。如果low 等于lenth的话，意味着最长递增序列的长度为 low + 1，所以length++。~~

### LIS

上面写的都是锤子呀，一年后再来看完全看不懂。只能看题解一步步跟着分析。做这道之前先去做了[最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/) ，它是本题的基础。

首先分析`最长递增子序列`的解法，题目大意是给定一个无序的数组找到最长的递增子序列长度。那怎么去找到这个序列呢？

> 如果我们要使上升子序列尽可能的长，则我们需要让序列上升得尽可能慢，因此我们希望每次在上升子序列最后加上的那个数尽可能的小。

我们用一个数组d去记录每个len 下的最后的那个数，并通过二分的策略不断往d 数组中更新更小的数，或者追加更大的数在数组末尾。二分更新最小的数是希望这个序列能够增长的尽可能的慢。 d[i]：表示数组中 长度为i的子序列中最末尾的元素。[此外题解中有求证 d 数组和其下标关系是严格递增的关系](https://leetcode-cn.com/problems/longest-increasing-subsequence/solution/zui-chang-shang-sheng-zi-xu-lie-by-leetcode-soluti/)。

以题解中的输入序列 [0, 8, 4, 12, 2] 为例：

第一步插入 0，d = [0]

第二步插入 8，d = [0, 8]

第三步插入 4，d = [0, 4]

第四步插入 12，d = [0, 4, 12]

第五步插入 2，d = [0, 2, 12]

细心的你想必注意到了，这个方法只能找到最大递增子序列的长度，并不能找到具体的最大递增子序列。 接下来的本题就和之前记录的一致了。对二维的 LIS 进行排序降维，注意对宽度递增，但是宽度相同时对高度需要递减。

举个🌰，[[2，3]，[1，4]，[1，3]，[1，5]] 都进行递增排序后得到 [[1，3]，[1，4]，[1，5]，[2，3]] 这个时候我们对 height 做 LIS会得到 [3, 4, 5]，可是注意哟 套娃是不允许放在宽度相同的套娃上的。为了防止这种情况，排序时宽度递增，宽度相同，高度递减。最后对高度 做 LIS 即可。

类似的题目还有[马戏团人塔](https://leetcode-cn.com/problems/circus-tower-lcci/)

做笔记时要认真呀, 做笔记时要认真呀, 做笔记时要认真呀.争取做一道会一类吧

时间复杂度： $O(nlogn)$

空间复杂度： $O(n)$

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