title: Linux epoll模型
date: 2019-3-3
tags: [Android，消息机制]
categories: Android开发艺术探索
description: MainActivity怎会如此流畅丝滑？
---
## 引言
举一个自己熟悉的栗子，读者，写者问题。当缓冲区满时，写者怎么办？缓冲区空的时候咋办尼？嗯~阻塞呀！没那么简单哦

## Linux下的I/O
在Linux下对引言中提出的问题，给出的解决办法是：
1. 阻塞：卡顿在那里，等待唤醒，与OS中一致
2. 轮询：不停询问，PV操作实现机制

流的概念，一个流可以是文件，socket，pipe等等可以进行I/O操作的内核对象。不管是文件，还是套接字，还是管道，我们都可以把他们看作流。
阻塞I/O的缺点。在阻塞I/O模式下，一个线程只能处理一个流的I/O事件。如果想要同时处理多个流，要么多进程(fork)，要么多线程(pthread_create)，很不幸这两种方法效率都不高。
那么讨论轮询，
```java
while true {
for i in stream[]; {
if i has data
read until unavailable
}
}
```
我们只要不停的把所有流从头到尾问一遍，又从头开始。这样就可以处理多个流了，但这样的做法显然不好，因为如果所有的流都没有数据，那么只会白白浪费CPU。这里要补充一点，阻塞模式下，内核对于I/O事件的处理是阻塞或者唤醒，而非阻塞模式下则把I/O事件交给其他对象（后文介绍的select以及epoll）处理甚至直接忽略。
为了避免CPU空转，可以引进了一个代理（select/poll代理）。这个代理比较厉害，可以同时观察许多流的I/O事件，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有I/O事件时，就从阻塞态中醒来，于是我们的程序就会轮询一遍所有的流（于是我们可以把“忙”字去掉了）。
```java
while true {
select(streams[])
for i in streams[] {
if i has data
read until unavailable
}
}
```
如果没有I/O事件产生，我们的程序就会阻塞在select处。但是依然有个问题，我们从select那里仅仅知道了，有I/O事件发生了，但却并不知道是那几个流（可能有一个，多个，甚至全部），我们只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。但是使用select，我们有O(n)的无差别轮询复杂度，同时处理的流越多，每一次无差别轮询时间就越长。

## epoll模型
`epoll 是一个可扩展的 Linux I/O 事件通知机制。`即一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的

epoll可以理解为event poll，不同于忙轮询和无差别轮询，epoll之会把哪个流发生了怎样的I/O事件通知我们。此时我们对这些流的操作都是有意义的。（复杂度降低到了O(k)）

使用epoll的时候，内核会维护一个就绪的链表，这个链表里面的东西就是"那些发生了I/O事件的流"。而这些文件描述符是如何被添加到就绪链表里面的呢？是通过注册的那些epoll在操作系统内核给IO事件注册回调函数回调函数实现的。epoll这个函数返回的只是那些发生了事件的文件描述符。
## Looper
终极问题：为什么Looper不卡咩？众所周知（我也不知道），app程序入口中为主线程准备好了消息队列，并开启了Looper轮询，而Looper.loop是一个死循环。那MainActivity不就卡了吗~

```java
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

```
先介绍一些基础知识：
进程：每个app运行时前首先创建一个进程，该进程是由Zygote fork出来的，用于承载App上运行的各种Activity/Service等组件。进程对于上层应用来说是完全透明的，这也是google有意为之，让App程序都是运行在Android Runtime。大多数情况一个App就运行在一个进程中，除非在AndroidManifest.xml中配置Android:process属性，或通过native代码fork进程。
线程：线程对应用来说非常常见，比如每次new Thread().start都会创建一个新的线程。该线程与App所在进程之间资源共享，从Linux角度来说进程与线程除了是否共享资源外，并没有本质的区别，都是一个task_struct结构体，`在CPU看来进程或线程无非就是一段可执行的代码`，CPU采用CFS调度算法，保证每个task都尽可能公平的享有CPU时间片。
### Android中为什么主线程不会因为Looper.loop()里的死循环卡死？
答： 主线程因为循环才得以存活，否则执行完一系列初始化语句后，就会径直退出

线程既然是一段可执行的代码，当可执行代码执行完成后，线程生命周期便该终止了，线程退出。而对于主线程，我们是绝不希望会被运行一段时间，自己就退出，那么如何保证能一直存活呢？`简单做法就是可执行代码是能一直执行下去的，死循环便能保证不会被退出，`例如，binder线程也是采用死循环的方法，通过循环方式不同与Binder驱动进行读写操作，当然并非简单地死循环，无消息时会休眠。但这里可能又引发了另一个问题，`既然是死循环又如何去处理其他事务呢？通过创建新线程的方式。`
真正会卡死主线程的操作是在回调方法onCreate/onStart/onResume等操作时间过长，会导致掉帧，甚至发生ANR，looper.loop本身不会导致应用卡死。
### 哪里有相关代码为这个死循环准备了一个新线程去运转？
会在进入死循环之前便创建了新binder线程，在代码ActivityThread.main()中：
```java
public static void main(String[] args) {
        ....

        //创建Looper和MessageQueue对象，用于处理主线程的消息
        Looper.prepareMainLooper();

        //创建ActivityThread对象
        ActivityThread thread = new ActivityThread(); 

        //建立Binder通道 (创建新线程)
        thread.attach(false);

        Looper.loop(); //消息循环运行
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

__thread.attach(false)；便会创建一个Binder线程（具体是指ApplicationThread，Binder的服务端，用于接收系统服务AMS发送来的事件），该Binder线程通过Handler将Message发送给主线程	。__

ActivityThread实际上并非线程，不像HandlerThread类，ActivityThread并没有真正继承Thread类，只是往往运行在主线程，该人以线程的感觉，其实承载ActivityThread的主线程就是由Zygote fork而创建的进程。

`主线程的死循环一直运行是不是特别消耗CPU资源呢？ `其实不然，这里就涉及到`Linux pipe/epoll机制`，简单说就是在主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()方法里，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。这里的唤醒机制用的epoll机制，`是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。` 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。
###  Activity的生命周期是怎么实现在死循环体外能够执行起来的

ActivityThread的内部类H继承于Handler，通过handler消息机制，简单说Handler机制用于同一个进程的线程间通信。

`Activity的生命周期都是依靠主线程的Looper.loop，当收到不同Message时则采用相应措施：`在H.handleMessage(msg)方法中，根据接收到不同的msg，执行相应的生命周期。 

比如收到`msg=H.LAUNCH_ACTIVITY`，则调用`ActivityThread.handleLaunchActivity()`方法，最终会通过反射机制，创建Activity实例，然后再执行`Activity.onCreate()`等方法；    再比如收到`msg=H.PAUSE_ACTIVITY`，则调用`ActivityThread.handlePauseActivity()`方法，最终会执行`Activity.onPause()`等方法。 上述过程，我只挑核心逻辑讲，真正该过程远比这复杂。

 __主线程的消息又是哪来的呢__ 当然是App进程中的其他线程通过Handler发送给主线程.看下图
从进程与线程间通信的角度，通过一张图加深自己对App运行过程的理解：
![App_run](/images/app_run_time.jpg)

__system_server进程是系统进程__ ，java framework框架的核心载体，里面运行了大量的系统服务，比如这里提供ApplicationThreadProxy（简称ATP），ActivityManagerService（简称AMS），这个两个服务都运行在system_server进程的不同线程中，由于ATP和AMS都是基于IBinder接口，都是binder线程，binder线程的创建与销毁都是由binder驱动来决定的。

__App进程则是我们常说的应用程序__ ，主线程主要负责Activity/Service等组件的生命周期以及UI相关操作都运行在这个线程； 另外，每个App进程中至少会有两个binder线程 ApplicationThread(简称AT)和ActivityManagerProxy（简称AMP），除了图中画的线程，其中还有很多线程，比如signal catcher线程等，这里就不一一列举。

Binder用于不同进程之间通信，由一个进程的Binder客户端向另一个进程的服务端发送事务，比如图中线程2向线程4发送事务；而handler用于同一个进程中不同线程的通信，比如图中线程4向主线程发送消息。

__结合图说说Activity生命周期，比如暂停Activity，流程如下：__ 

1. 线程1的AMS中调用线程2的ATP；（由于同一个进程的线程间资源共享，可以相互直接调用，但需要注意多线程并发问题）
2. 线程2通过binder传输到App进程的线程4
3. 线程4通过handler消息机制，将暂停Activity的消息发送给主线程；
4. 主线程在looper.loop()中循环遍历消息，当收到暂停Activity的消息时，便将消息分发给ActivityThread.H.handleMessage()方法，再经过方法的调用，最后便会调用到Activity.onPause()，当onPause()处理完后，继续循环loop下去。

## 参考链接
https://www.zhihu.com/question/34652589
https://www.zhihu.com/search?type=content&q=linux%20epoll
