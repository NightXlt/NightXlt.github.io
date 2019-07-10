title:  Find the contiguous subarray which has the largest sum
date: 2019-1-8
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.
 ## Examples
``` java
Example 1:
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

## Solution
　　这道题曾在剑指Offer上看过解法，有两种解法
　　1. DP，做过最直观的DP啦~_~   $$ dp[i] =\left\{
\begin{aligned}
dp[i - 1] + nums[i] && \ f(i - 1)  >= 0 \&\& i != 0 \\
nums[i] && \ f[i - 1] < 0 ||  i == 0 \\
\end{aligned}
\right.
$$
　　2. 找规律，记录累加和，当累加和小于0时，说明前面的子数组已构成一个最大和（听着有点别扭哈）。令累加和等于当前值，重新开始累加。

## Code

```java
/**DP
     * @param nums
     * @return the max sum of sub array
     */
     public int maxSubArray(int[] nums) {
        int dp[] = new int[nums.length];
        dp[0] = nums[0];
        int maxSum = dp[0];
        for (int i = 1; i < nums.length; i++) {
            if (dp[i - 1] > 0) {
                dp[i] = dp[i - 1] + nums[i];
            } else {
                dp[i] = nums[i];
            }
            maxSum = Math.max(dp[i], maxSum);
        }
        return maxSum;
    }

```
```java
/**Find the law 
     * @param nums
     * @return the max sum of sub array
     */
    public int maxSubArray(int[] nums) {
        int maxSum = Integer.MIN_VALUE;
        int curSum = 0;
        for (int i = 0; i < nums.length; i++) {
            curSum = (curSum <= 0) ? nums[i] : (nums[i] + curSum);
            maxSum = Math.max(maxSum, curSum);
        }
        return maxSum;
    }

```