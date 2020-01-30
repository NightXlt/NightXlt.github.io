title: Otto源码剖析
date: 2019-2-26
tags: [Android,事件总线,源码解析]
categories: Android开发艺术探索
description: 　　
---
## 有意思的单例
```java
public enum LovelyBus {

  INSTANCE; // 优雅的单例

  private Bus mBus;

  LovelyBus() {

    mBus = new Bus();
  }

  public void register(Object object) {

    mBus.register(object);
  }

  public void unregister(Object object) {

    mBus.unregister(object);
  }

  public void post(Object event) {

    mBus.post(event);
  }

}

```
## Bus的构造函数
```java
  public Bus() {
    this(DEFAULT_IDENTIFIER);
  }

  public Bus(String identifier) {
    this(ThreadEnforcer.MAIN, identifier);
  }

  public Bus(ThreadEnforcer enforcer) {
    this(enforcer, DEFAULT_IDENTIFIER);
  }

  public Bus(ThreadEnforcer enforcer, String identifier) {
    this(enforcer, identifier, HandlerFinder.ANNOTATED);
  }
  
  Bus(ThreadEnforcer enforcer, String identifier, HandlerFinder handlerFinder) {
    this.enforcer =  enforcer;
    this.identifier = identifier;
    this.handlerFinder = handlerFinder;
  }
```
ThreadEnforcer.MAIN表示默认在主线程调度事件。handlerFinder(HandlerFinder.ANNOTATED)则是查找订阅者和订阅者。

## register
```java
  public void register(Object object) {
    if (object == null) {
      throw new NullPointerException("Object to register must not be null.");
    }
    enforcer.enforce(this);

    Map<Class<?>, EventProducer> foundProducers = handlerFinder.findAllProducers(object);
```
获取这个被注册对象的所有生产者方法，可以对应于多个事件的生产者方法。一个被注册对象，同一个事件只能注册一个生产者方法。如@Produce注解的方法只能有一个参数为String的方法。再看接下来的register方法

```java
//key是事件的class对象
    Map<Class<?>, EventProducer> foundProducers = handlerFinder.findAllProducers(object);
    for (Class<?> type : foundProducers.keySet()) {
          
     //下面几行代码用来检查，事件是否已经注册过了，一个类一个事件只能注册一次
      final EventProducer producer = foundProducers.get(type);
      EventProducer previousProducer = producersByType.putIfAbsent(type, producer);
      //checking if the previous producer existed
      if (previousProducer != null) {
        throw new IllegalArgumentException("Producer method for type " + type
          + " found on type " + producer.target.getClass()
          + ", but already registered by type " + previousProducer.target.getClass() + ".");
      }
      //检查一下注册这个事件的订阅者是否存在，存在就回调一下订阅者
      Set<EventHandler> handlers = handlersByType.get(type);
      if (handlers != null && !handlers.isEmpty()) {
        for (EventHandler handler : handlers) {
          dispatchProducerResultToHandler(handler, producer);
        }
      }
    }
```
dispatchProducerResultToHandler(handler, producer);
具体实现就是通过反射去调用producer生产出事件，把事件传递给handler，再通过反射回调注册的方法。
```java
    Map<Class<?>, Set<EventHandler>> foundHandlersMap = handlerFinder.findAllSubscribers(object);
    for (Class<?> type : foundHandlersMap.keySet()) {
	////type是事件的class对象
      Set<EventHandler> handlers = handlersByType.get(type);
      if (handlers == null) {
        //concurrent put if absent
        Set<EventHandler> handlersCreation = new CopyOnWriteArraySet<EventHandler>();
        handlers = handlersByType.putIfAbsent(type, handlersCreation);
        if (handlers == null) {
            handlers = handlersCreation;
        }
      }
      final Set<EventHandler> foundHandlers = foundHandlersMap.get(type);
      if (!handlers.addAll(foundHandlers)) {
        throw new IllegalArgumentException("Object already registered.");
      }
    }

    for (Map.Entry<Class<?>, Set<EventHandler>> entry : foundHandlersMap.entrySet()) {
      Class<?> type = entry.getKey();
      EventProducer producer = producersByType.get(type);
      if (producer != null && producer.isValid()) {
        Set<EventHandler> foundHandlers = entry.getValue();
        for (EventHandler foundHandler : foundHandlers) {
          if (!producer.isValid()) {
            break;
          }
          if (foundHandler.isValid()) {
            dispatchProducerResultToHandler(foundHandler, producer);
          }
        }
      }
    }
```
获取所有订阅者，如果有与订阅者对应的发布者，则触发对应的订阅者处理事件。
## post
```java
public void post(Object event) {
    if (event == null) {
      throw new NullPointerException("Event to post must not be null.");
    }
    enforcer.enforce(this);
//返回这个event的所有继承关系链的所有class对象（父类class对象和自己）
    Set<Class<?>> dispatchTypes = flattenHierarchy(event.getClass());

    boolean dispatched = false;
	//遍历集合对even的所有类的相关注册方法加入队列
    for (Class<?> eventType : dispatchTypes) {
      Set<EventHandler> wrappers = getHandlersForEventType(eventType);

      if (wrappers != null && !wrappers.isEmpty()) {
        dispatched = true;
        for (EventHandler wrapper : wrappers) {
		 //把需要回调处理的订阅者方法的包装类塞进当前线程处理的队列中去。queue是一个ThreadLocal对象
          enqueueEvent(event, wrapper);
        }
      }
    }

    if (!dispatched && !(event instanceof DeadEvent)) {
      post(new DeadEvent(this, event));
    }
//取出队列中的对象执行订阅者方法
    dispatchQueuedEvents();
  }
```
## unregister
```java
public void unregister(Object object) {
    if (object == null) {
      throw new NullPointerException("Object to unregister must not be null.");
    }
    enforcer.enforce(this);
//发布者取消注册
    Map<Class<?>, EventProducer> producersInListener = handlerFinder.findAllProducers(object);
    for (Map.Entry<Class<?>, EventProducer> entry : producersInListener.entrySet()) {
      final Class<?> key = entry.getKey();
      EventProducer producer = getProducerForEventType(key);
      EventProducer value = entry.getValue();

      if (value == null || !value.equals(producer)) {
        throw new IllegalArgumentException(
            "Missing event producer for an annotated method. Is " + object.getClass()
                + " registered?");
      }
      producersByType.remove(key).invalidate();
    }
//订阅者取消注册
    Map<Class<?>, Set<EventHandler>> handlersInListener = handlerFinder.findAllSubscribers(object);
    for (Map.Entry<Class<?>, Set<EventHandler>> entry : handlersInListener.entrySet()) {
      Set<EventHandler> currentHandlers = getHandlersForEventType(entry.getKey());
      Collection<EventHandler> eventMethodsInListener = entry.getValue();

      if (currentHandlers == null || !currentHandlers.containsAll(eventMethodsInListener)) {
        throw new IllegalArgumentException(
            "Missing event handler for an annotated method. Is " + object.getClass()
                + " registered?");
      }

      for (EventHandler handler : currentHandlers) {
        if (eventMethodsInListener.contains(handler)) {
          handler.invalidate();
        }
      }
      currentHandlers.removeAll(eventMethodsInListener);
    }
  }
```
