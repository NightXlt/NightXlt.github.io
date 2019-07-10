title: EventBus源码解析
date: 2019-3-3
tags: [Android,开源框架]
categories: 源码解析
description: 　　
---
转自[EventBus解析](https://www.bookstack.cn/read/android_interview/android-open-source-framework-EventBus.md)
## 简析EventBus
EventBus是一个Android端优化的publish/subscribe消息总线，简化了应用程序内各组件间、组件与后台线程间的通信。
作为一个消息总线主要有三个组成部分：

事件（Event）：可以是任意类型的对象。通过事件的发布者将事件进行传递。

事件订阅者（Subscriber）：接收特定的事件。

事件发布者（Publisher）：用于通知 Subscriber 有事件发生。可以在任意线程任意位置发送事件。
![eventbus_resolve](/images/eventbus_simple_resolve.png)
事件的发布者（Publisher）将事件（Event）通过post()方法发送。EventBus内部进行处理，找到订阅了该事件（Event）的事件订阅者（Subscriber）。然后该事件的订阅者（Subscriber）通过onEvent()方法接收事件进行相关处理（关于onEvent()在EventBus 3.0中有改动，下面详细说明）。

## demo
注册订阅
``` java
EventBus.getDefault().register(this);
```
事件处理
``` java
@Subscribe(threadMode = ThreadMode.MainThread)
    public void  onNewsEvent(NewsEvent event){
        String message = event.getMessage();
        mTv_message.setText(message);
    }
```
发布事件
``` java
 EventBus.getDefault().post(new NewsEvent("我是来自SecondActivity的消息，你好！"));
```

## Register
进入getDefault()
``` java
static volatile EventBus defaultInstance;
public static EventBus getDefault() {
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
                }
            }
        }
        return defaultInstance;
    }
```
通过上述代码可以得知，getDefault()中通过双检查锁（DCL）机制实现了EventBus的单例机制，获得了一个默认配置的EventBus对象。
在了解register()之前，先要了解一下EventBus中的几个关键的成员变量。方便对下面内容的理解。
``` java
/* Map<订阅事件, 订阅该事件的订阅者集合> */
private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;
/* Map<订阅者, 订阅事件集合> */
private final Map<Object, List<Class<?>>> typesBySubscriber;
/* Map<订阅事件类类型,订阅事件实例对象>. */
private final Map<Class<?>, Object> stickyEvents;
```
下面看具体的register()中执行的代码。
``` java
    public void register(Object subscriber) {
	//订阅者类型
        Class<?> subscriberClass = subscriber.getClass();
		 //获取订阅者全部的响应函数信息（即上面的onNewsEvent()之类的方法）
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
		 //循环每一个事件响应函数，执行 subscribe()方法，更新订阅相关信息
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
```
register()
1. 获取订阅者的类类型
2. 通过SubscriberMethodFinder类来解析订阅者类,获取所有的响应函数集合
3. 遍历订阅函数,执行 subscribe()方法，更新订阅相关信息

继续看subscribe()方法。
subscribe 函数分三步
第一步：
``` java
        //获取订阅的事件类型
        Class<?> eventType = subscriberMethod.eventType;
       //获取订阅该事件的订阅者集合
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
      //把通过register()订阅的订阅者包装成Subscription 对象
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
        //订阅者集合为空，创建新的集合，并把newSubscription 加入
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<Subscription>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            //集合中已经有该订阅者，抛出异常。不能重复订阅
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }
       //把新的订阅者按照优先级加入到订阅者集合中。
        synchronized (subscriptions) {
            int size = subscriptions.size();
            for (int i = 0; i <= size; i++) {
                if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                    subscriptions.add(i, newSubscription);
                    break;
                }
            }
        }
```
第二步：
``` java
        //根据订阅者，获得该订阅者订阅的事件类型集合
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
       //如果事件类型集合为空，创建新的集合，并加入新订阅的事件类型。
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<Class<?>>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
       //如果事件类型集合不为空，加入新订阅的事件类型
        subscribedEvents.add(eventType);
```
第三步：
``` java
//该事件是stick=true。
if (subscriberMethod.sticky) {
            //响应订阅事件的父类事件
            if (eventInheritance) {
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                //循环获得每个stickyEvent事件
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    //是该类的父类
                    if (eventType.isAssignableFrom(candidateEventType)) {
                         //该事件类型最新的事件发送给当前订阅者。
                        Object stickyEvent = entry.getValue();
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
```
1. 通过subscriptionsByEventType得到该事件类型所有订阅者信息队列，根据优先级将当前订阅者信息插入到订阅者队列subscriptionsByEventType中；
2. 在typesBySubscriber中得到当前订阅者订阅的所有事件队列，将此事件保存到队列typesBySubscriber中，用于后续取消订阅；
3. 检查这个事件是否是 Sticky 事件，如果是则从stickyEvents事件保存队列中取出该事件类型最后一个事件发送给当前订阅者。

以下是订阅的具体流程图：
![eventbus_register](/images/eventbus_register.png)

## unRegister
``` java
public synchronized void unregister(Object subscriber) {
    // 获取该订阅者所有的订阅事件类类型集合.
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    if (subscribedTypes != null) {
        for (Class<?> eventType : subscribedTypes) {
            unsubscribeByEventType(subscriber, eventType);
        }
        // 从typesBySubscriber删除该<订阅者对象,订阅事件类类型集合>
        typesBySubscriber.remove(subscriber);
    } else {
        Log.e("EventBus", "Subscriber to unregister was not registered before: "
                + subscriber.getClass());
    }
}
private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
    // 获取订阅事件对应的订阅者信息集合.
    List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions != null) {
        int size = subscriptions.size();
        for (int i = 0; i < size; i ++) {
            Subscription subscription = subscriptions.get(i);
            // 从订阅者集合中删除特定的订阅者.
            if (subscription.subscriber == subscriber) {
                subscription.active = false;
                subscriptions.remove(i);
                i --;
                size --;
            }
        }
    }
}
```
unregister()方法比较简单，主要完成了subscriptionsByEventType以及typesBySubscriber两个集合的同步。
## Post
``` java
public void post(Object event) {
    // 获取当前线程的Posting状态.
    PostingThreadState postingState = currentPostingThreadState.get();
    // 获取当前线程的事件队列.
    List<Object> eventQueue = postingState.eventQueue;
    //将当前事件添加到其事件队列
    eventQueue.add(event);
    //判断新加入的事件是否在分发中
    if (!postingState.isPosting) {
        postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            // 循环处理当前线程eventQueue中的每一个event对象.
            while (!eventQueue.isEmpty()) {
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            // 重置postingState一些标识信息.
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```
post 函数会首先得到当前线程的 post 信息 PostingThreadState，其中包含事件队列，将当前事件添加到其事件队列中，然后循环调用 postSingleEvent 函数发布队列中的每个事件。
``` java
private void postSingleEvent(Object event, PostingThreadState postingState) {
   //分发事件的类型
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
   //响应订阅事件的父类事件
    if (eventInheritance) {
        //找出当前订阅事件类类型eventClass的所有父类的类类型和其实现的接口的类类型
        List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h ++) {
            Class<?> clazz = eventTypes.get(h);
            //发布每个事件到每个订阅者
            subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
        }
    } else {
        subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
    }
........
}
```
调用 postSingleEventForEventType 函数发布每个事件到每个订阅者。
``` java
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState,
                                            Class<?> eventClass) {
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        // 获取订阅事件类类型对应的订阅者信息集合.(register函数时构造的集合)
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription : subscriptions) {
            postingState.event = event;
            postingState.subscription = subscription;
            boolean aborted = false;
            try {
                // 发布订阅事件给订阅函数
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            if (aborted) {
                break;
            }
        }
        return true;
    }
    return false;
}

```
调用 postToSubscription 函数向每个订阅者发布事件。
postToSubscription 函数中会判断订阅者的 ThreadMode，从而决定在什么 Mode 下执行事件响应函数。这里就不贴源码了。后续还牵着到反射以及线程调度问题，这里就不展开了。
```java
    private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case MAIN_ORDERED:
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }

```
其中mainThreadPoster是一个Handler。
下面是具体的post的流程图
![eventbus_post](/images/eventbus_post.png)
