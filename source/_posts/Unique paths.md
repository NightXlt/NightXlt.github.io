title: Unique paths
date: 2019-1-28
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).How many possible unique paths are there?
 ## Examples
``` java
Example 1:
Input: m = 3, n = 2
Output: 3
Explanation:
From the top-left corner, there are a total of 3 ways to reach the bottom-right corner:
1. Right -> Right -> Down
2. Right -> Down -> Right
3. Down -> Right -> Right
```
```java
Example 2:
Input: m = 7, n = 3
Output: 28
```
## Solution
　　模拟做出来了，但超时了。试图剪枝，仍无果。网上参考了别人的思路。
  　　1. 数学法：机器人总共走m+n-2步，其中m-1步往下走，n-1步往右走，本质上就是一个组合问题，也就是从m+n-2个不同元素中每次取出min(m-1, n -1)个元素的组合数。根据组合的计算公式，可以写代码来求解即可。有个小技巧；$$ {m \choose n}  =  {n - m \choose n}$$ 
　　故而可取较小的m计算阶乘，降低时间复杂度
  　　2. 动态规划法：维护一个二维数组dp初始化为1，其中dp[i][j]表示到当前位置不同的走法的个数，然后可以得到递推式为:$$ dp[i][j] = dp[i - 1][j] + dp[i][j - 1]$$，这里为了节省空间，我们使用一维数组dp逐行刷新模拟，即$$dp[i] = dp[i] + dp[i - 1]$$

## Code

```java
//OverTime
 class Solution {
       int count = 0;
    int row, col;

    public int uniquePaths(int m, int n) {
        if (m == 1 && n == 1) {
            return 1;
        }
        col = m;
        row = n;
        path(0, 0);
        return count;
    }

    public void path(int m, int n) {
        if (col - 1 == m && row - 1 == n) {
            count++;
            return;
        }
        if (col - 1 == m && row - 1 != n)
            path(m, n + 1);
        else if (row - 1 == n && col - 1 != m)
            path(m + 1, n);
        else {
            path(m + 1, n);
            path(m, n + 1);
        }
    }

}
```

```java
//Math
class Solution {
     public int uniquePaths(int m, int n) {
        if (m == 1 && n == 1) {
            return 1;
        }
        double numerator = 1, denominator = 1;
        int small = (m > n) ? n - 1 : m - 1;
        for (int i = 1; i <= small; i++) {
            denominator *= i;
            numerator *= m + n - 1 - i;
        }
        return (int) (numerator / denominator);
    }
}
```

```java
class Solution{
    public int uniquePaths(int m, int n) {
        int[] a = new int[n];
        for (int i = 0; i < a.length; i++) {
            a[i] = 1;
        }
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                a[j] += a[j - 1];
            }
        }
        return a[n - 1];
    }
}
```