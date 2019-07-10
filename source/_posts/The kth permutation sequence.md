title: The kth permutation sequence.
date: 2019-1-2
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### The set [1,2,3,...,n] contains a total of n! unique permutations.By listing and labeling all of the permutations in order, we get the following sequence for n = 3:　"123"，"132"，"213"，"231"，"312"，"321"；Given n and k, return the kth permutation sequence.Note:Given n will be between 1 and 9 inclusive.Given k will be between 1 and n! inclusive.
 ## Examples
``` java
Example 1:
Input: n = 3, k = 3
Output: "213"
```
```java
Example 2:
Input: n = 4, k = 9
Output: "2314"
```

## Solution
　　首先需将k--转换为从0开始计数，index = k％(n-1)!，得到的index就是当前剩余数组的index，如此就可以得到对应的元素。如此递推直到数组中没有元素结束。实现中我们要维护一个数组来记录当前的元素，每次得到一个元素加入结果数组，然后从剩余数组中移除（一定要移，而不是get），因此空间复杂度是O(n)。时间上总共需要n个回合，而每次删除元素如果是用数组需要O(n),所以总共是O(n^2)。


## Code

```java
 class Solution {
        public String getPermutation(int n, int k) {
        k--;
        List<Integer> integers = new ArrayList<>();
        int factorial = 1;
        for (int i = 1; i <= n; i++) {
            integers.add(i);
            factorial *= i;
        }
        factorial /= n;
        int count = n - 1;
        StringBuilder sb = new StringBuilder();
        while (count >= 0) {
            sb.append(integers.remove(k / factorial));
            k %= factorial;
            if (count != 0) {
                factorial /= count;
            }
            count--;
        }
        return sb.toString();
    }

}
```