title: The permutation of string
date: 2018-12-28
tags: [Leetcode,字节跳动]
categories: algorithm
description: Given two strings s1 and s2, write a function to return true if s2 contains the permutation of s1.
---
## 题目描述
  ### Given two strings s1 and s2, write a function to return true if s2 contains the permutation of s1. In other words, one of the first string's permutations is the substring of the second string.
 ## Examples
``` java
Example 1:
Input:s1 = "ab" s2 = "eidbaooo"
Output:True
Explanation: s2 contains one permutation of s1 ("ba").
```
```java
Example 2:
Input:s1= "ab" s2 = "eidboaoo"
Output: False
```
```java
Example 3:
Input:s1= "abc" s2 = "abdc"
Output: True

```

## Solution

1. 一开始找不到排列和子序列的对应关系；可以通过记录两个字符串中字符出现次数countS1[],countS2[]来比较，但这样会引入一个问题可能两个子序列出现次数相等，然而出现次序不一致。为了解决这一问题，在此思路上引入滑动窗口（没错，就是计网中的流量控制中的滑动窗口协议）。

2. 在本题中令countS2[]作为发送窗口，countS1[]作为接收窗口。先比较前s1的长度len1内的元素，再循环比较剩余s2的长度len2-len1个元素。总共比较次数len2-len1+1次。但这样写的话循环内需要加一层判断当前发送窗口是否越界，增加了时间复杂度。

3. 可优化为先单独长度为len1的元素判断。循环（接收窗口滑动一个单位，再判断是否相等），这时第i个字符小时， i + len1个字符出现。 直至比较完毕或出现不等退出循环。这样比较剩余元素可确保发送窗口不会越界。（执行顺序注意下）切记__循环内套if要三思啊，炒鸡炒鸡炒鸡耗时的。__

## Code

```java
//an effective algorithm
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int len1 = s1.length();
        int len2 = s2.length();
        if (len1 > len2) {
            return false;
        }
        int countS1[] = new int[26];
        int countS2[] = new int[26];
        for (int i = 0; i < len1; i++) {
            countS1[s1.charAt(i) - 'a']++;
            countS2[s2.charAt(i) - 'a']++;
        }
        if (isEquals(countS1, countS2))
            return true;
        for (int i = 0; i < len2 - len1; i++) {

            //sliding window
            countS2[s2.charAt(i) - 'a']--;
            countS2[s2.charAt(i + len1) - 'a']++;
            if (isEquals(countS1, countS2))//Judging after sliding
                return true;
        }
        return false;
    }

    public boolean isEquals(int c1[], int c2[]) {
        for (int i = 0; i < c1.length; i++) {
            if (c1[i] != c2[i]) {
                return false;
            }
        }
        return true;
    }

}
```

```java
//a relative slower algorithm
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int len1 = s1.length();
        int len2 = s2.length();
        if (len1 > len2) {
            return false;
        }
        int countS1[] = new int[26];
        int countS2[] = new int[26];
        for (int i = 0; i < len1; i++) {
            countS1[s1.charAt(i) - 'a']++;
            countS2[s2.charAt(i) - 'a']++;
        }
        for (int i = 0; i < len2 - len1 + 1; i++) {
  			 if (isEquals(countS1, countS2))//Judging after sliding
               	 	return true;
            //sliding window
			if(i< len2 - len1){
			 	 countS2[s2.charAt(i) - 'a']--;
                 countS2[s2.charAt(i + len1) - 'a']++;
			}
        }
        return false;
    }

    public boolean isEquals(int c1[], int c2[]) {
        for (int i = 0; i < c1.length; i++) {
            if (c1[i] != c2[i]) {
                return false;
            }
        }
        return true;
    }

}
```