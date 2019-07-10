title: UTF-8 validation
date: 2019-1-9
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### A character in UTF8 can be from 1 to 4 bytes long, subjected to the following rules:For 1-byte character, the first bit is a 0, followed by its unicode code.For n-bytes character, the first n-bits are all one's, the n+1 bit is 0, followed by n-1 bytes with most significant 2 bits being 10.

| Char. number range(hexadecimal) | UTF-8 octet sequence (binary) | 
| ------------------------------- | ----------------------------- | --- | --- |
| 0000 0000-0000 007F             | 0xxxxxxx                      | 
| 0000 0080-0000 07FF             | 110xxxxx 10xxxxxx             |  
| 0000 0800-0000 FFFF             | 1110xxxx 10xxxxxx 10xxxxxx    |     
| 0001 0000-0010 FFFF             | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

 is how the UTF-8 encoding would work:
 ## Examples
``` java
Example 1:
data = [197, 130, 1], which represents the octet sequence: 11000101 10000010 00000001.

Return true.
It is a valid utf-8 encoding for a 2-bytes character followed by a 1-byte character.
```
```java
Example 2:
data = [235, 140, 4], which represented the octet sequence: 11101011 10001100 00000100.

Return false.
The first 3 bits are all one's and the 4th bit is 0 means it is a 3-bytes character.
The next byte is a continuation byte which starts with 10 and that's correct.
But the second continuation byte does not start with 10, so it is invalid.
```
## Solution
　　看着题目发了好久的呆，实在读不懂题目，UTF-8 编码的意思是读懂了，可是感觉给的样例对应不起来。看着解析的代码一步步分析终于得出意思了。样例中数组的每个数值x的二进制若以0开头，
则x表示一个unicode码。若以110开头，接下来的一个数字都必须以10开头。若以1110开头，接下来的两个数字都必须以10开头。若以11110开头，接下来的三个数字都必须以10开头。

## Code
```java
     public boolean validUtf8(int[] data) {
        int count = 0;
        for (int i = 0; i < data.length; i++) {
            if (count == 0) {
                if(data[i] >> 5 == 0b110)
                    count = 1;
                else if(data[i] >> 4 == 0b1110)
                    count = 2;
                else if(data[i] >> 3 == 0b11110)
                    count = 3;
                else if(data[i] >> 7 != 0)
                    return false;

            } else {
                if(data[i] >> 6 != 0b10)
                    return false;
                count--;
            }
        }
        return count == 0;
    }
```