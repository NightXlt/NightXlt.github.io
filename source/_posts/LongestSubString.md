title: The longest substring without repeating characters.
date: 2018-12-24 00:00:00
tags: [Leetcode,字节跳动]
categories: algorithm
description: Given a string, find the length of the longest substring without repeating characters.
---


## 题目描述
  ### Given a string, find the length of the longest substring without repeating characters.
 ## Examples
``` java
Example 1:

Input: "abcabcbb" 
Output: 3
Explanation: The answer is "abc", with the length of 3. 
```
```java
Example 2:

Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1. 
```
```java
Example 3:

Input: "pwwkew"
Output: 3 Explanation: 
The answer is "wke", with the length of 3. Note that the answer must

```

## Solution

首先自己意识到了用HashSet进行添加非重复元素，但出现元素重复时应该删除哪一个却不确定。当出现重复元素时画图分析可知，当前记录String的leftIndex至rightIndex区间内子序列可能并非最长，这时应删除l当前leftIndex所指向元素。让leftIndex++,重新开始计算。


## Code

```java
  public int lengthOfLongestSubstring(String s) {
        int lIndex = 0, rIndex = 0;
        int n = s.length();
        Set<Character> characterSet = new HashSet<>();
        int maxSubStringLength = 0;
        while (rIndex < n && lIndex < n) {
            char c = s.charAt(rIndex);
            if (!characterSet.contains(c)) {
                characterSet.add(c);
                rIndex++;
                maxSubStringLength = Math.max(maxSubStringLength, rIndex - lIndex);
            } else {
                characterSet.remove(s.charAt(lIndex++));
            }
        }
        return maxSubStringLength;
    }
```