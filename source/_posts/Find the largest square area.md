title: Find the largest square area
date: 2019-1-7
mathjax: true
tags: [Leetcode,字节跳动]
categories: algorithm
description:  Find the largest square containing only 1's and return its area.
---
## Problem description
  ### Given a 2D binary matrix filled with 0's and 1's, find the largest square containing only 1's and return its area.
 ## Examples
``` java
Example 1:
Input: 

1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0

Output: 4
```
## Solution
　　方向是确定的，DP，但总是想不出来状态方程出来。图是个好东西。
  ![dp1](/images/Find the largest square area dp1.jpg )
  dp[i][j]表示以matrix[i][j]为右下角的正方形最大边长（1.保证一定是正方形 2.保证边长为最大边长）。为什么可以做到保证为正方形呢？当matrix[2][2] = 1时，从图中观察可知当dp[1][1]、dp[1][2] 、dp[2][1]但凡只要有一个为0，必然构不成正方形。
  所以状态方程为：
  $$ dp[i][j] =\left\{
\begin{aligned}
min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1]) + 1 && \ matrix[i][j] = 1 \\
0 && \ matrix[i][j] = 0 \\
\end{aligned}
\right.
$$

## Code

```java
 class Solution {
        public int maximalSquare(char[][] matrix) {
        if (matrix.length == 0) {
            return 0;
        }
        int max = Integer.MIN_VALUE;
        int row = matrix.length;
        int column = matrix[0].length;
        int f[][] = new int[row][column ];
        for (int i = 0; i < row; i++) {
            if (matrix[i][0] == '1') {
                f[i][0] = 1;
                max = 1;
            }
        }
        for (int i = 0; i < column; i++) {
            if (matrix[0][i] == '1') {
                f[0][i] = 1;
                max = 1;
            }
        }
        for (int i = 1; i < row; i++) {
            for (int j = 1; j < column; j++) {
                if (matrix[i][j] == '1') {
                    f[i][j] = Math.min(f[i - 1][j - 1], Math.min(f[i][j - 1], f[i - 1][j])) + 1;
                    max = Math.max(max, f[i][j]);
                }
            }
        }
        return max * max;
    }

}
```