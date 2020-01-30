title: Message Mechanism
date: 2019-1-30
tags: [Android]
categories: Android开发艺术探索
description: 　　
---
## Message Queue
　　MessageQueue主要包含两个操作。1. 插入enqueueMessage(Message message);  2. 读取 消息next()
  MessageQueue虽然名字中含有Queue，但实际上并没有用队列实现。而是使用Message自带的单链表。
  ### enqueueMessage
  ```java
      boolean enqueueMessage(Message msg, long when) {//单链表插入
     	 ...
        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }

  ```
  ### next（）
  ```java
      Message next() {//获取下一条msg，MessageQueue中没有msg则阻塞轮询Poll。
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }

  ```
  
  ## Looper
  　　Looper:轮询器，它会不停从MessageQueue中查看是否有新消息，如果有则立即处理，否则就一直阻塞。Handler必须配合Looper使用，之所以平常使用没用创建Looper，是因为在UI线程ActivityThread创建时就会初始化Looper。在Looper初始化的同时也会初始化MessageQueue.
	
```java
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```
```java
    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```
loop方法是一个死循环，唯一可以跳出循环的方式是MessageQueue的next()返回了null，而只有当Looper被设置为退出时，MessageQueue的next()才会返回null。若MessageQueue的next()返回一条新消息，Looper就会调用msg.target.dispatchMessage(msg);处理。msg.target是Handler。故Handler发送的消息被Looper原封不动的交由Handler的dispatchMessager（）处理。原封不动指的是dispatchMessage（）会在创建Handler的线程下运行回调handleMessage().

## Handler

![send_message](/images/message_mechanism.PNG)
1. 在A线程中创建Looper（创建MessageQueue）， Handler
2. Handler在B线程中发送消息，消息会插入到A线程中的MessageQueue中
3. Looper从MessageQueue中取出消息回调Handler的dispatchMessage处理消息
4. dispatchMessage中会调用Handler的handleMessage方法


### sendMessage()
```java
/*
Handler将消息传到MessageQueue中，Looper通过MessageQueue。next()获取消息，Looper再将消息交由Handler的dispatchMessage()处理
*/
    public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
	
public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
	    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

```
### dispatchMessage()

```java
    /**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {//msg.callback  is a runnable
            handleCallback(msg);//Run callback(Handler.post(Runnable r))
        } else {
            if (mCallback != null) {//mCallback is a interface that has handdleMessage method.(We can init Handler by invoking Handler handler = new Handler(Callback call))
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

```
  
## 关于Handler.postdelayed()一个消息阻塞中，又来个消息
首先在next()方法中消息A处于阻塞状态（没有消息了或者只有Delay的消息），会把mBlocked这个变量标记为true
```java
Message next() {
    for (;;) {
        ...
        if (msg != null) {
            ...
        } else {
            // Got a message.
            mBlocked = false;
            ...
        }
        ...
    }
    ...
    if (pendingIdleHandlerCount <= 0) {
        // No idle handlers to run.  Loop and wait some more.
        mBlocked = true;
        continue;
    }
｝    ...
```
当新消息B到来时，进入enqueueMessage（）
```java
    boolean enqueueMessage(Message msg, long when) {
...
msg.markInUse();
msg.when = when;
Message p = mMessages;
boolean needWake;
if (p == null || when == 0 || when < p.when) {
    // New head, wake up the event queue if blocked.
    msg.next = p;
    mMessages = msg;
    needWake = mBlocked;
} else {
    ...
}
...
// We can assume mPtr != 0 because mQuitting is false.
    if (needWake) {
          nativeWake(mPtr);
    }
...
}
```
新到消息B的延迟时间 < 阻塞消息A的延迟时间。则会将B放在队列头部，A放在B的后面执行。needWake置为true。调用nativeWake()方法唤醒线程

整个调用流程：
1. postDelay()一个10秒钟的Runnable A、消息进队，MessageQueue调用nativePollOnce()阻塞，Looper阻塞；
2. 紧接着post()一个Runnable B、消息进队，判断现在A时间还没到、正在阻塞，把B插入消息队列的头部（A的前面），然后调用nativeWake()方法唤醒线程；
3. MessageQueue.next()方法被唤醒后，重新开始读取消息链表，第一个消息B无延时，直接返回给Looper；
4. Looper处理完这个消息再次调用next()方法，MessageQueue继续读取消息链表，第二个消息A还没到时间，计算一下剩余时间（假如还剩9秒）继续调用nativePollOnce()阻塞；
5. 直到阻塞时间到或者下一次有Message进队；

题外话：MessageQueue会根据post delay的时间排序放入到链表中，链表头的时间小，尾部时间最大。因此能保证时间Delay最长的不会block住时间短的。当每次post message的时候会进入到MessageQueue的next()方法，会根据其delay时间和链表头的比较，如果更短则，放入链表头，并且看时间是否有delay，如果有，则block，等待时间到来唤醒执行，否则将唤醒立即执行。