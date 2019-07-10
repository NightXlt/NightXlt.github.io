title:  Find the single one number
date: 2019-1-13
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a non-empty array of integers, every element appears twice except for one. Find that single one.Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?
 ## Examples
``` java
Example 1:
Input: [2,2,1]
Output: 1
```
```java
Example 2:
Input: [4,1,2,1,2]
Output: 4
```


## Solution
　　异或概念：
  　

| &&      | 一假则假，短路 |
| ------- | -------------- |
| &          | 全判断                |
| &#124;&#124;           | 一真则真，短路 |
| &#124;      | 全判断         |
| ^(异或) | 相异为真       |
  
　　剑指Offer的一道题，采用异或的方法。异或可以实现在不使用临时变量的情况下，交换两个值。
```java
a = a ^ b
b = a ^ b // a ^ b ^ b = a ^ 0 = a
a = a ^ b // a ^ b ^ a = b ^ 0 = b
```
## Code

```java
 class Solution {
    public int singleNumber(int[] nums) {
        if (nums == null || nums.length == 0) {
            return -1;
        }
        int result = 0;
        for (int i = 0; i < nums.length; i++) {
            result = result ^ nums[i];
        }
        return result;
    }

}
```