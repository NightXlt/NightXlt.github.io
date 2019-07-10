title: HandlerThread和IntentService源码解析
date: 2019-2-24
tags: [Android,线程]
categories: Android开发艺术探索
description: 关于IntentService,我想记得的
---
## HandlerThread
类如其名 类 = Handler + Thread.HandlerThread继承自Thread,且HandlerThread包含Handler成员变量。进入run（）
```java
@Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
```
在run()内创建了一个Looper，并开启了轮询，这注定了HandlerThread继承自Thread，毕竟主线程已经由了MainLooper,一个线程只能有一个Looper,否则就会抛出异常。此外run()是无限循环的，当明确不再使用HandlerThread时需要调用quit()或quitSafely()

## IntentService
IntentService是继承于Service并处理异步请求的一个类。IntentService继承了Service且是一个抽象类，故必须创建其子类进行使用。
IntentService用于后台执行任务，执行完毕它就会自动停止.封装了HandlerThread和Handler的特殊Service。进入IntentService的onCreate()
```java
@Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
```
初始化HandlerThread.并用HandlerThread的Looper构造了mServiceHandler.mServiceHandler发送的消息最终会在HandlerThread中执行。故mServiceHandler始终运行在子线程。IntentService每启动一次，它就会调用onStartCommand()处理Intent.而onStartCommand（）又会调用.onStart()。
其实mServiceHandler可直接当成HandlerThread中的Handler。

```java
@Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
```
进入mServiceHandler的handleMessage()
```java
private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
```
onHandleIntent（）是一个抽象方法，需要我们自己在其中定义耗时操作。stopSelf（）会停止服务；如果存在多个后台任务。onHandlerIntent（）执行完最后一个才会调用stopSelf().每执行一个后台任务就需要启动一次IntentService。而IntentService则通过向HandlerThread发送消息执行任务。而HandlerThread中的Looper按顺序排队执行任务。
```java
public class LocalIntentService extends IntentService {

  public LocalIntentService() {
    super("sdd");
  }

  /**
   * Creates an IntentService.  Invoked by your subclass's constructor.
   *
   * @param name Used to name the worker thread, important only for debugging.
   */

  public LocalIntentService(String name) {
    super(name);
  }

  @Override protected void onHandleIntent(@Nullable Intent intent) {
    String action = intent.getAction();
    Log.i("TAG", "onHandleIntent: " + action);
    SystemClock.sleep(3000);
    if ("night".equals(action)) {
      Log.i("TAG", "handle " + action);
    }
  }

  @Override public void onDestroy() {
    Log.i("TAG", "onDestroy: ");
    super.onDestroy();
  }
}
 @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_personal_view);
    Intent service = new Intent(this, LocalIntentService.class);
    service.setAction("night");
    startService(service);
    service.setAction("sdf");
    startService(service);
    service.setAction(",poop");
    startService(service);
  }
```