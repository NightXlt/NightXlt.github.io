title:   Divide Two Integers
date: 2020-1-31
tags: [Leetcode]
categories: algorithm

description:  不使用乘除，mod 运算来实现除法　
---

## Problem description

https://leetcode-cn.com/problems/divide-two-integers/

给定两个数，不能使用乘除法，mod 运算。只能使用 Int 进行存储。而且当为 Int.MIN_VALUE(- 2^31) / -1返回 Int.MAX_VALUE(2^31 - 1) 

Return the quotient after dividing dividend by divisor.

The integer division should truncate toward zero.

Note:

- Both dividend and divisor will be 32-bit signed integers.
- The divisor will never be 0.
- Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−2^31,  2^31 − 1]. For the purpose of this problem, assume that your function returns 2^31 − 1 when the division result overflows.


## Solution

不能使用乘除，mod求商。那只能使用位运算。位运算中左移右移会牵涉到*2操作。通过`二分法`进行迫近被除数，2^(n0 + n1 + n2 + n3+ ....) * divisor = dividend，我们需要做的就是逐一找到 n0, n1 等值。 比如 dividend = 10, divisor = 3可以拆为

3 *（2^1 + 2^0）+ 余数 等于 10，约等于是因为我们求得是`商`。除数 * 商是可能小于被除数的。余数是满足 < divisor 的。

我们先找到divisor * 2 ^n0,这个 n0是所有幂中最大的一个，其最接近于 dividend。比如 dividend = 10, divisor = 3。那n0 = 1。再让 dividend = 10 - 3 * 2 = 4，再找到新的 dividend 符合的n1.直至 divisor > dividend 终止，即余数小于除数。将找到的 n0，n1 等值计算得出 2的幂次方和返回。

大致思路如上，具体有些边界处理的问题，比如Int 的最大值为 2^31 - 1, 进行左移时，会出现 2 ^31越界情况。故将其均转为负数统一处理。在转为负数后，原来大的数就变小了，5 > 3, -5 < -3. if 判断里牵涉到 dividend 和 diviso比较r都需要变大小与号。

此外为了防止 divisor * 2^n0 < Int.MIN_VALUE溢出，需要加一层兜底。那这层兜底应该怎么加呢？ 难道是这么判断

if (divisor shl 1 < Int.MIN_VALUE ) {}，非也，如果 divisor 快要溢出了，再左移一位就会变为正数。Int.MIN_VALUE左移一位为 0. 那何不将 Int.MIN_VALUE 变小一点再进行比较呢？如 if (divisor < Int.MIN_VALUE shr 1) {}。值得注意的是这里没有等于号没有等于号没有等于号。我们允许divisor * 2^n == dividend，但不能越界。

此外在从 Int.MIN_VALUE 变号为正号时需要特殊处理。



## Code

```java
class Divide {
    fun divide(dividend: Int, divisor: Int): Int {
        if (divisor == 0) {
            throw ArithmeticException("divide zero exception")
        }
        val sign = (dividend > 0) xor (divisor > 0) //记录两个数是否同号
        var dividend = dividend
        var divisor = divisor
        if (dividend > 0) {
            dividend = -dividend
        }
        if (divisor > 0) {
            divisor = -divisor
        }
        var count = 0
        while (dividend <= divisor shl 1) {
            if (divisor < Int.MIN_VALUE shr 1) {
                break
            }
            count++
            divisor = divisor.shl(1)

        }
        var res = 0
        while (count >= 0) {
            if (divisor >= dividend) {
                res += (-1).shl(count)
                dividend -= divisor
            }
            count--
            divisor = divisor.shr(1)
        }
        if (sign) { //异号直接返回
            return res
        }
        if (res <= Int.MIN_VALUE) { //同号 * -1前需要进行边界处理
            return Int.MAX_VALUE
        }
        return -res
    }
}
```