title: Reverse the string
date: 2018-12-29
tags: [Leetcode,字节跳动]
categories: algorithm
description: Reverse the string word by word.
---
## 题目描述
  ### Given an input string, reverse the string word by word.A word is defined as a sequence of non-space characters.Input string may contain leading or trailing spaces. However, your reversed string should not contain leading or trailing spaces.You need to reduce multiple spaces between two words to a single space in the reversed string.
 ## Examples
``` java
Example 1:
Input: "the sky is blue",
Output: "blue is sky the".
```
```java
Example 2:
Input: "",
Output: "".
```
```java
Example 3:
Input: "   ",
Output: "".

```

## Solution
  思路是以“ ”作为分隔符，再通过流转换消去所有“”，但这样太过耗时，总共跑了70ms，可以尝试通过拼接字符串的方式降低时间复杂度。
  还有个Split坑，以前用都没感觉到，这次空格数多了发现和预期有差异。进入Split源码
 ```java
 int resultSize = list.size();
            if (limit == 0) {//controls the number of times the pattern is applied  
			// limit = n :  The pattern is applied n-1 times,limit = 0 :the pattern will be applied as many times as possible
                while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
                    resultSize--;
                }
            }
			String[] result = new String[resultSize];
            return list.subList(0, resultSize).toArray(result);
 ```
   默认调用的split（regex）,它会调用split(regex,0).所以在list末尾处若出现 “”，就将ressultSize--;最后再截取list返回array。
## Code

```java
 public String reverseWords(String s) {
        if (null == s || s.equals("")) {
            return "";
        }
        String[] strings = s.split(" ",-1);//全部split,不进行尾处理
        StringBuilder sb = new StringBuilder();
        for (int i = strings.length - 1; i >= 0; i--) {
            s = strings[i].trim();
            if(s.length() > 0) sb.append(' ');
            sb.append(s);
        }
        return sb.toString().trim();
    }
```