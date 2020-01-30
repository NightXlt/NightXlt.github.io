title: Find longest palindromic substring
date: 2019-1-15
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.
 ## Examples
``` java
Example 1:
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```
```java
Example 2:
Input: "cbbd"
Output: "bb"
```
## Solution
　　参考至[https://blog.csdn.net/qq_32354501/article/details/80084325](https://blog.csdn.net/qq_32354501/article/details/80084325)
  　　1. 采取从中心向外扩散方法，根据子串长度分为奇数和偶数情况
  　　2. 遍历每个除最后一个位置的字符index(字符位置)，奇数：初始low = 初始high = index，low和high均不超过原字符串的下限和上限；判断low和high处的字符是否相等，相等则low--、high++（偶数：初始high = 初始low+1 = index + 1）；
  　　3. 每次low与high处的字符相等时，都将当前最长的回文子串长度与high-low+1比较。后者大时，记录两端索引
  　　4. 重复执行2）、3），直至high-low+1 等于原字符串长度或者遍历到最后一个字符，取当前截取到的回文子串，该子串即为最长的回文子串。

## Code

```java
 class Solution {
       public static int sMaxLength, leftIndex, rightIndex;

    public String longestPalindrome(String s) {
        sMaxLength = 0;
        leftIndex = 0;
        rightIndex = 0;
        if (s == null || s.length() <= 1) {
            return s;
        }
        for (int i = 0; i < s.length(); i++) {
            findLongestPalindromic(s, i, i);//odd
            findLongestPalindromic(s, i, i + 1);//even
        }
        return s.substring(leftIndex, rightIndex);
    }

    public void findLongestPalindromic(String s, int low, int high) {
        while (low >= 0 && high <= s.length() - 1) {
            if (s.charAt(low) == s.charAt(high)) {
                int len = high - low + 1;
                if (len > sMaxLength) {
                    sMaxLength = len;
                    leftIndex = low;
                    rightIndex = high + 1;
                }
                low--;//Bear in mind that changing index
                high++;
            } else {
                break;
            }
        }
    }


}
```