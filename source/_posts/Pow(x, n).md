title:  Pow(x, n)
date: 2019-2-1
tags: [Leetcode,腾讯]
categories: algorithm
description: 右移，右移讷
---
## Problem description
  ### Implement pow(x, n), which calculates x raised to the power n (xn).
 ## Note
  -100.0 < x < 100.0
n is a 32-bit signed integer, within the range [−2^31, 2^31 − 1]
 ## Examples
``` java
Example 1:
Input: 2.00000, 10
Output: 1024.00000
```
```java
Example 2:
Input: 2.10000, 3
Output: 9.26100
```
```java
Example 3:
Input: 2.00000, -2
Output: 0.25000
Explanation: 2-2 = 1/22 = 1/4 = 0.25
```

## Solution
这道题有个坑的地方，当 x = 2时，n = -2^31时。超时了，在这一行代码上超时了，n >> = 1;  自己用了右移运算反而超时了.百思不得其解。当求一个数的负数次方时，可以采取1 / pow(x, -n);转换为求倒数。这么一转换对于这个样例刚好出问题了。出在哪了呢？1. 当对-2^31取负数时，该数仍旧是负数。因为int的最大值是2^31 - 1超过了最大值。 2. 对负数进行右移>>高位补符号位。这样得到的结果就会出错，而且会超时。
解决方案：1. 采用 /        2. 采取无符号右移>>>
-5 / 2 = -2，5 / 2 = 2。这表明除二是向零取整
-5 （1011）>> 1 =（1101） -3，5 >> 1 = 2。这补码表示表明右移一位是向下取整

## Code

```java
 class Solution {

    public double myPow(double x, int n) {
        if (n < 0) {
            return 1 / pow(x, -n);
        } else {
            return pow(x, n);
        }
    }

    public double pow(double base, int exponent) {
        if (exponent == 0) {
            return 1;
        }
        if (exponent == 1 || base == 1) {
            return base;
        }
        double res = 1;
        while (exponent != 0) {
            if ((exponent & 0x1) == 1) {
                res *= base;
            }
            base *= base;
            exponent >>>= 1;
        }
        return res;
    }

}
```