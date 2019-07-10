title:  Gray code
date: 2019-1-27
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### The gray code is a binary numeral system where two successive values differ in only one bit.Given a non-negative integer n representing the total number of bits in the code, print the sequence of gray code. A gray code sequence must begin with 0.
 ## Examples
``` java
Example 1:
Input: 2
Output: [0,1,3,2]
Explanation:
00 - 0
01 - 1
11 - 3
10 - 2
```
```java
Example 2:
Input: 0
Output: [0]
Explanation: We define the gray code sequence to begin with 0.
             A gray code sequence of n has size = 2n, which for n = 0 the size is 20 = 1.
             Therefore, for n = 0 the gray code sequence is [0].
```
## Solution
　　两种做法，一种是回溯，另一种是找规律（对于编程技术提高然并卵）。 
　　找规律格雷编码与二进制的关系： 从最右边一位起，依次将每一位与左边一位异或(XOR)，作为对应格雷码该位的值，最左边一位不变(相当于左边是0)。如 6(0110) 与3( 0011) 异或得到格雷编码0101.
　　回溯法：也有点找规律的意思，长度为n的格雷编码 = 长度为 n - 1的格雷编码每个编码前加0,再逆序对每个编码前加1.如长度为1的编码为0 ，1.而长度为2的编码为00，01，11，10.

## Code
```java
class Solution {
    public List<Integer> grayCode(int n) {
        n = 1 << n;
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            list.add(i ^ (i >> 1));
        }
        return list;
    }
}
```
```java
 class Solution {
    public List<Integer> grayCode(int n) {
        List<Integer> list = new ArrayList<>();
        dfs(list, 1, n);
        return list;
    }

    void dfs(List<Integer> list, int cur, int n) {
        if (n == 0) {
            list.add(0);
            return;
        }
        if (cur > n) {
            return;
        }
        if (cur == 1) {
            list.add(0);
            list.add(1);
        } else {
            int len = list.size();
            for (int i = 0; i < len; i++) {
                list.add((int) (list.get(len - i - 1) + Math.pow(2, cur - 1)));
            }
        }
        dfs(list, cur + 1, n);
    }
}
```