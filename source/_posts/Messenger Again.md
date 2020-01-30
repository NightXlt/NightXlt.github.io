title: Messenger Again
date: 2019-1-20
tags: [Android,IPC]
categories: Android开发艺术探索
description: Messenger对aidl做了些什么？
---
## Messenger 与 AIDL的前世今生
　　Messenger这个信使呀，每天东奔西走的传递各个Message对象。咦，是不是好熟悉。传递Message的信使，哎呀不还有个Handler老哥吗？关于消息机制会在下一章消息机制进行解析。说了也巧，Messenger中Handler也插足了。但不是负责传递消息。而是负责接受处理。简单的说就是受了。而Messenger负责传。Messenger的底层实现是aidl。但又不同于aidl。Messenger只能串行传递消息，即一次一个消息。而aidl可以并发执行。
  
  ## 传递原理
  ![Messenger](/images/Messenger.png)
  盗自Hyman大佬。
  前提：C/S都维护着一个Messenger和Handler。
  1. 客户端连接建立时获取到服务器Messenger发送客户端消息请求时，并将自己的Messenger 作为回复Messenger（msg.replyTo）传入。
  2. 服务端的Handler收到消息后会取出客户端的Messenger.发送服务端消息回复。并将自己的Messenger 作为回复Messenger（msg.replyTo）传入。
  3. 客户端的Handler收到消息处理。

局部代码：
```java
//Client
  private Messenger mReplyMessenger = new Messenger(new MessengerHandler());
  private static class MessengerHandler extends Handler{
    @Override public void handleMessage(Message msg) {
      switch (msg.what) {
        case MessengerService.MSG_FROM_SERVICE:
          Log.i("NightXLT", "Receive: " + msg.getData().getString("reply"));
          break;
        default:
          super.handleMessage(msg);
          break;
      }
    }
  }
 private ServiceConnection mServiceConnection = new ServiceConnection() {
    @Override public void onServiceConnected(ComponentName name, IBinder service) {
      mService = new Messenger(service);
      Message msg = Message.obtain(null, MessengerService.MSG_FROM_CLIENT);
      Bundle data = new Bundle();
      data.putString("msg", "sdfsdfsdf");
      /*Book boo = new Book(0, "the spirit of Chinese");
      data.putParcelable("Book", boo);*/
      msg.setData(data);
      msg.replyTo = mReplyMessenger;
      try {
        mService.send(msg);
      } catch (RemoteException e) {
        Log.i("NightXLT", "happen e");
        e.printStackTrace();
      }
    }
```
```java
//Service
public Messenger messenger = new Messenger(new MyHandler());
public class MyHandler extends Handler{
@Override
public void handleMessage(Message msg) {
Log.i("Ser---TAG", "msg::"+msg.arg1+"want :"+msg.getData().getString("msg"));
Messenger messenger = msg.replyTo;
Message message = Message.obtain(null, MSG_FROM_SERVICE);
Bundle bundle = new Bundle();
bundle.putString("reply", "嗯,你的消息我已经收到,稍后回复你!");
message.setData(bundle);
try {
messenger.send(message);
} catch (RemoteException e) {
e.printStackTrace();
}
super.handleMessage(msg);
}
}
```