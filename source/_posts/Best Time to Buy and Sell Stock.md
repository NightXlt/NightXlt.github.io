title: Best Time to Buy and Sell Stock
date: 2019-1-6
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Say you have an array for which the ith element is the price of a given stock on day i.If you were only permitted to complete at most one transaction (i.e., buy one and sell one share of the stock), design an algorithm to find the maximum profit.Note that you cannot sell a stock before you buy one.
 ## Examples
``` java
Example 1:
Input: [7,1,5,3,6,4]
Output: 5
```
```java
Example 2:
Input: [7,6,4,3,1]
Output: 0
```
## Solution
　　这道贪心的题关键在于找到前面n个的最小，不需要用Treeset排序。只需要记录前i个的最小。以及后面的多笔交易也与之类似。

## Code

```java
//Single transaction
 class Solution {
      public int maxProfit(int[] prices) {
        if (prices.length == 0 || prices.length == 1) {
            return 0;
        }
        int maxProfit = 0;
        int min = prices[0];
        for (int i = 1; i < prices.length; i++) {
            min = Math.min(min, prices[i]);
            maxProfit = Math.max(maxProfit, prices[i] - min);
        }
        return maxProfit;
    }
}
```

```java
//Multiple transaction
class Solution {
     public int maxProfit(int[] prices) {
        if (prices.length == 0 || prices.length == 1) {
            return 0;
        }
        int maxProfit = 0;
        int min = prices[0];
        for (int i = 1; i < prices.length; i++) {
            if (prices[i] - min > 0) {
                maxProfit += prices[i] - min;
            }
            min = prices[i];
        }
        return maxProfit;
    }


}
```