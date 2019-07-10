title: The min stack
date: 2019-1-6
tags: [Leetcode,字节跳动]
categories: algorithm
description: 　　
---
## Problem description
  ### Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.
 ## Examples
``` java
Example 1:
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> Returns -3.
minStack.pop();
minStack.top();      --> Returns 0.
minStack.getMin();   --> Returns -2.
```

## Solution
　　逻辑没问题，但就是怎么就AC不了。还是那个老问题。以前是list里添加list要new。这道题是同样问题。List添加对象时，也要每次new，每次靠 = 修改对象再添加。这样list内的对象会都一样。
## Code

```java
 class MinStack {

    Stack<Integer> mStack;
    Stack<Integer> mMinStack;

    public MinStack() {
        mStack = new Stack<>();
        mMinStack = new Stack<>();
    }

    public void push(int x) {
        mStack.push(x);
        if (mMinStack.size() == 0 || x < mMinStack.peek()) {
            mMinStack.push(x);
        } else {
            mMinStack.push(mMinStack.peek());
        } 
    }

    public void pop() {
        if (!mStack.isEmpty()) {
            mStack.pop();
            mMinStack.pop();
        }
    }

    public int top() {
        return mStack.peek();
    }

    public int getMin() {
        return mMinStack.peek();
    }
}

```