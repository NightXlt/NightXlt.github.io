title:  Determine if it is a power of two
date: 2019-1-23
tags: [Leetcode,字节跳动]
categories: algorithm
description: Given an integer, write a function to determine if it is a power of two.
---
## Problem description
  ### Given an integer, write a function to determine if it is a power of two.
 ## Examples
``` java
Example 1:
Input: 1
Output: true 
Explanation: 20 = 1
```
```java
Example 2:
Input: 16
Output: true
Explanation: 24 = 16
```
```java
Example 3:
Input: 218
Output: false
```

## Solution
　　这道题在剑指Offer上的“二进制中1的个数”的封装。首先可以确定的是2的幂的二进制中最左端只有1，其余为0。若如此，可将n = （n - 1）& n，这样会将n二进制中最右端的1赋值为0.。若n为2的幂的话，进行与运算后，n 会等于0。否则不等于 0.
## Code

```java
 class Solution {
    public boolean isPowerOfTwo(int n) {
        if (n <= 0) {
            return false;
        }
        return ((n - 1) & n) == 0;
    }
}
```