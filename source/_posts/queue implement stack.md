title: 利用队列实现栈
date: 2019-1-31
tags: [Leetcode,腾讯]
categories: algorithm
description: 　　
---
## Problem description
  ### Implement the following operations of a stack using queues.

push(x) -- Push element x onto stack.
pop() -- Removes the element on top of the stack.
top() -- Get the top element.
empty() -- Return whether the stack is empty.
 ## Examples
``` java
Example 1:
MyStack stack = new MyStack();

stack.push(1);
stack.push(2);  
stack.top();   // returns 2
stack.pop();   // returns 2
stack.empty(); // returns false
```
## Solution
　　起初采取是剑指Offer上常规的做法，插入时根据turn的值插到对应队列，pop和peek时将n - 1个元素转移到另一个队列。再返回最后一个元素。参考了别人的代码。原来可以不用turn记录插入弹出队列。如果可以维护一个出队顺序 = 出栈顺序的队列的话那多方便啊。可这是队列呀，不一样对吧。所以用两个栈维护入栈顺序，每添加一个元素x前，先将queue1中的元素先转移到queue2中，再添加x（这时queue1中只有一个元素x）。最后将queue2中的元素转移回来，这样queue1的出队列顺序即是出栈顺序。降低了pop、peek时间复杂度。整个过程入栈和出栈最终都是在queue1中，queue2只是作为一个中转栈。
## Code

```java
 class MyStack {

  Queue<Integer> q1, q2;

    /**
     * Initialize your data structure here.
     */
    public MyStack() {
        q1 = new ArrayDeque<>();
        q2 = new ArrayDeque<>();
    }

    /**
     * Push element x to the back of queue.
     */
    public void push(int x) {
        while (!q1.isEmpty()) {
            q2.offer(q1.poll());
        }
        q1.offer(x);
        while (!q2.isEmpty()) {
            q1.offer(q2.poll());
        }
    }

    /**
     * Removes the element from in front of queue and returns that element.
     */
    public int pop() {
        return q1.isEmpty() ? -1 : q1.poll();
    }

    /**
     * Get the front element.
     */
    public int top() {
        return q1.isEmpty() ? -1 : q1.peek();
    }

    /**
     * Returns whether the queue is empty.
     */
    public boolean empty() {
        return q2.isEmpty() && q1.isEmpty();
    }
}
```