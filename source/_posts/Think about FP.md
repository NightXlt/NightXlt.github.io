title: 灵通关于Function thinking的一些思考
date: 2019-1-17
tags: [Function Programming]
categories: Java
description: 哇，好爽~_~
---
## 前言
本文源自对Neal Ford所著的函数式编程思维一书的一些思考记录。自己也曾写过RX全家桶，Java8的函数式表达式。直观感受是简洁。只为简洁Java单独用一个版本推出这个表达式吗？在Java8之前Java是不支持函数式编程的。Java要实现函数式编程必须要借助Function Java第三方库。那么为何众多语言都加入了对函数式编程的支持呢？函数式编程的背后又是什么呢？

## 思维转变

> 函数式编程中粒度最小的重用单元是函数（一等公民），并具备值不可变性

命令式编程风格是按照“程序是一系列改变状态的命令”来建模，即解决问题的步骤。如典型的for循环。而函数式编程则是基于表达式和变换，即数据的映射，更类似一种数学建模的思想。用map、filter这些高阶函数把我们解放出来，让我们站在更高的抽象层次上去考虑问题 ，把问题看得更清楚。


## 值不可变

> OOP通过封装不确定因素使代码让人理解。而函数式编程通过减少使代码让人理解。                                   ——Michael Feathers

不确定因素：变量值，函数值等状态。OOP在竭力控制这些状态得出正确的结果。而函数式编程(FP)则恰恰相反。FP则是尽可能消灭可变的状态。如果一门语言没有那么多容易掉进去的坑，那么开发者就不会容易犯错。所以有函数式编程中数据不可变性（immutable data）一说，多有的变量只可以赋值一次，变量不可变，如果想改变变量就创建一个新的变量。
## 缓求值
缓求值的好处：昂贵的运算只有到了绝对必要的时候才执行；可以建立无限大的集合，只要一接到请求就一直送出元素；按缓求值的方式使用map、filter等，可以产生更高效的代码。
```java
package springapp.domain;

import java.util.ArrayList;
import java.util.List;

import static java.util.stream.Collectors.toList;

/**
 * Created by Night on 2018/12/28.
 * Desc:
 */

public class Solution {
    private int asd = 3;
    
    void print() {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        asd = 4;
        System.out.println(list.parallelStream()
                .filter(n -> n > 1)
                .map(s -> s = s * asd)
                .collect(toList()));
    }
    public static void main(String[] args) {
       new Solution().print();
    }
}

```
在lambda表达式中引入了状态变量asd。在map映射时其并未真正执行乘法操作。而是直至调用collect（）方才执行。这也就是FP中的缓求值的概念。在这其中我们交出了对状态的管理权让运行时去管理状态。推迟执行是FP中一个及其重要的思想。无论缓求值亦或闭包都是基于这个思想。