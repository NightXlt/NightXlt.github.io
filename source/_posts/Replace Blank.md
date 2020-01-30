title: Replace Blank
date: 2019-1-29
tags: [字符串,剑指Offer]
categories: algorithm
description: 　　
---
## Problem description
  ### 将字符串中每个空格替换成“can”
  
 ## Examples
``` java
Example 1:
Input:sdf sddsf sef
Output:sdfcansddsfcansef
```

## Solution
　　首先遍历一遍数组得到空格数目。每替换一个空格，字符串长度增加2.替换后的字符串长度为原来的长度乘以2.
  　　从字符串的末尾开始向头移动复制替换。indexOfOriginal指向原始字符串的末尾，indexOfNew指向替换之后的字符串末尾。遇到空格时，indexOfNew前插入can.indexOfOrigin--；否则将indexOfOriginal所指向的值复制进入indexOfNew指向位置。直至indexOfOriginal < 0 或者indexOriginal >= indexOfNew结束。

## Code

```java
 void ReplaceBlank(char[] s, int length) {
        if (s == null && s.length == 0) {
            return;
        }
        int numberOfBlank = 0;
        for (int i = 0; i < length; i++) {
            if (s[i] == ' ') {
                numberOfBlank++;
            }
        }
        int newLength = length + numberOfBlank * 2;
		if（newLength > length）{
		　　return;
		}
        int indexOfOriginal = length;
        int indexOfNew = newLength;

        while (indexOfOriginal >= 0 && indexOfNew > indexOfOriginal) {
            if (s[indexOfOriginal] == ' ') {
                s[indexOfNew--] = 'n';
                s[indexOfNew--] = 'a';
                s[indexOfNew--] = 'c';
            } else {
                s[indexOfNew--] = s[indexOfOriginal];
            }
            --indexOfOriginal;
        }
        System.out.println("");
    }
```