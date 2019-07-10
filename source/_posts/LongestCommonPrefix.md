title:  The longest common prefix string 
date: 2018-12-27
tags: [Leetcode,字节跳动]
categories: algorithm
description: Write a function to find the longest common prefix string amongst an array of strings
---
## 题目描述
  ### Write a function to find the longest common prefix string amongst an array of strings.If there is no common prefix, return an empty string "".
 ## Examples
``` java
Example 1:
Input: ["flower","flow","flight"]
Output: "fl"
```
```java
Example 2:
Input: ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
```
```java
Example 3:
Input: ["aa","a"]
Output: "a"

```

## Solution
 个人思路是先比较出头两个字符串的最长公共前缀longestCommonPrefix，如果为零，则直接返回 “”，否则将longestCommonPrefix与剩余字符串比较直至longestCommonPrefix为空或与其余字符串比较完毕。想法是不错，写到一半就一百行代码啦！！！  
 嗨呀，感觉方法有问题。横向比较行不通转成纵向比较。因为strs可看为一个二维数组。以第一行为参照，逐行比较同列元素直至一列中出现不相等情况退出循环。<front color="#00923">**值得注意的Example3的情况，若第一行元素并非strs最短字符串时需要防止访问字符串越界**</front>
## Code

```java
   public  String longestCommonPrefix(String[] strs) {

        if (strs.length == 0) {
            return "";
        }
        char c;
        int rowLength;
        int length = strs[0].length();
        for (int i = 0; i < length; i++) {
            c = strs[0].charAt(i);//c:第0行i列字符
            for (int j = 1; j < strs.length; j++) {
                rowLength = strs[j].length();
                if (i == rowLength || c != strs[j].charAt(i)) {//i == rowLength: secure  IndexInOfBounds
                    return strs[0].substring(0, i);
                }
            }
        }
        return strs[0];
    }
```