title: Climbing a stair
date: 2019-1-27
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### You are climbing a stair case. It takes n steps to reach to the top.Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?Note: Given n will be a positive integer.
 ## Examples
``` java
Example 1:
Input: 2
Output: 2
Explanation: There are two ways to climb to the top.
1. 1 step + 1 step
2. 2 steps
```
```java
Example 2:
Input: 3
Output: 3
Explanation: There are three ways to climb to the top.
1. 1 step + 1 step + 1 step
2. 1 step + 2 steps
3. 2 steps + 1 step
```
## Solution
　　剑指Offer上的青蛙跳台阶翻版，即斐波那契数列。

## Code

```java
 class Solution {
       public int climbStairs(int n) {
        int[] res = new int[]{0, 1, 2};
        if (n < 3) {
            return res[n];
        }
        int fibMinTwo = 1,fibMinOne = 2;
        int fibN = 0;
        for (int i = 2; i < n; i++) {
            fibN = fibMinOne + fibMinTwo;
            fibMinTwo = fibMinOne;
            fibMinOne = fibN;
        }
        return fibN;
    }
}
```