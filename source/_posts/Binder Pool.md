title: Binder Pool
date: 2019-1-28
tags: [Android,IPC]
categories: Android开发艺术探索
description: 实现一个service管理多个aidl
---
![binder_pool](/images/binder_pool.png)
从图中可知，每个aidl在Binder连接池中都有一个对应的Binder实例。在BinderPool中提供一个根据code查询对应Binder的query方法。
1. 服务端创建三个aidl文件
```java
// ISecurityCenter.aidl
interface ISecurityCenter {
  String encrypt(String content);
  String decrypt(String password);
}

  ```
  ```java
// ICompute.aidl
interface ICompute {
    int add(int a, int b);
}

```
  ```java
// IBinderPool.aidl
interface IBinderPool {
   IBinder queryBinder(int binderCode);
}

```

2. 服务端实现对应Binder实例
```java
public class SecurityCenterImpl extends ISecurityCenter.Stub {

  private static final char SECRET_CODE = '^';

  @Override public String encrypt(String content) throws RemoteException {
    char[] chars = content.toCharArray();
    for (int i = 0; i < chars.length; i++) {
      chars[i] ^= SECRET_CODE;
    }
    return new String(chars);
  }

  @Override public String decrypt(String password) throws RemoteException {
    return encrypt(password);
  }
}
```
```java
public class ComputeImpl extends ICompute.Stub {
  @Override public int add(int a, int b) throws RemoteException {
    return a + b;
  }
}
```
```java
public class BinderPool {
  public static final int BINDER_NONE = -1;
  public static final int BINDER_COMPUTE = 0;
  public static final int BINDER_SECURITY_CENTER = 1;

  private Context mContext;
  private IBinderPool mBinderPool;
  private static volatile BinderPool sInstance;
  private CountDownLatch mConnectBinderPoolCountDownLatch;
  //sync operation,Prevent the conflicts of binding.

  private ServiceConnection mBinderPoolConnection = new ServiceConnection() {
    @Override public void onServiceConnected(ComponentName name, IBinder service) {
      mBinderPool = IBinderPool.Stub.asInterface(service);
      try {
        mBinderPool.asBinder().linkToDeath(mBinderDeathRecipient, 0);//Prevent unexpected exit
      } catch (RemoteException e) {
        e.printStackTrace();
      }
      mConnectBinderPoolCountDownLatch.countDown();
    }

    @Override public void onServiceDisconnected(ComponentName name) {

    }
  };
  private IBinder.DeathRecipient mBinderDeathRecipient = new IBinder.DeathRecipient() {
    @Override public void binderDied() {
      Log.i("TAG", "binderDied: ");
      mBinderPool.asBinder().unlinkToDeath(mBinderDeathRecipient, 0);
      mBinderPool = null;
      connectBinderPoolService();
    }
  };
  private BinderPool(Context context) {
    mContext = context.getApplicationContext();
    connectBinderPoolService();//Begin conncect service
  }
  public static BinderPool getInsance(Context context) {
    if (sInstance == null) {
      synchronized (BinderPool.class) {
        if (sInstance == null) {
          sInstance = new BinderPool(context);
        }
      }
    }
    return sInstance;
  }
  private synchronized void connectBinderPoolService() {
    mConnectBinderPoolCountDownLatch = new CountDownLatch(1);//Only one client can bind service
    Intent service = new Intent(mContext, BinderPoolService.class);
    mContext.bindService(service, mBinderPoolConnection, Context.BIND_AUTO_CREATE);
    try {
      mConnectBinderPoolCountDownLatch.await();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }

  public IBinder queryBinder(int bindCode) {
    IBinder binder = null;
    if (mBinderPool != null) {
      try {
        binder = mBinderPool.queryBinder(bindCode);
      } catch (RemoteException e) {
        e.printStackTrace();
      }
    }
    return binder;
  }
  public static class BinderPoolImpl extends IBinderPool.Stub {
    public BinderPoolImpl() {
      super();
    }

    @Override public IBinder queryBinder(int binderCode) throws RemoteException {
      IBinder binder = null;
      switch (binderCode) {
        case BINDER_SECURITY_CENTER:
          binder = new SecurityCenterImpl();
          break;
        case BINDER_COMPUTE:
          binder = new ComputeImpl();
          break;
        default:
          break;
      }
      return binder;
    }
  }
}

```
3. 服务端返回Binder连接池
```java
//Service
public class BinderPoolService extends Service {


  private Binder mBinderPool = new BinderPool.BinderPoolImpl();

  @Override
  public void onCreate() {
    Log.i("TAG", "service:onCreate: ");
    super.onCreate();
  }

  @Override
  public IBinder onBind(Intent intent) {
    Log.d("TAG", "onBind");
    return mBinderPool;
  }

  @Override
  public void onDestroy() {
    super.onDestroy();
  }

}
```

4.客户端通过BinderPool获得对应Binder。再从Binder中获取已被服务器实现了的接口。
```java
public class BinderPoolActivity extends AppCompatActivity {
  private ISecurityCenter mSecurityCenter;
  private ICompute mCompute;

  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    new Thread(new Runnable() {

      @Override
      public void run() {
        doWork();
      }
    }).start();
  }

  private void doWork() {
    BinderPool binderPool = BinderPool.getInsance(BinderPoolActivity.this);
    IBinder securityBinder = binderPool
        .queryBinder(BinderPool.BINDER_SECURITY_CENTER);
    ;
    mSecurityCenter = (ISecurityCenter) SecurityCenterImpl
        .asInterface(securityBinder);//Get implemented interface
    Log.d("TAG", "visit ISecurityCenter");
    String msg = "helloworld-安卓";
    System.out.println("content:" + msg);
    try {
      String password = mSecurityCenter.encrypt(msg);
      System.out.println("encrypt:" + password);
      System.out.println("decrypt:" + mSecurityCenter.decrypt(password));
    } catch (RemoteException e) {
      e.printStackTrace();
    }

    Log.d("TAG", "visit ICompute");
    IBinder computeBinder = binderPool
        .queryBinder(BinderPool.BINDER_COMPUTE);
    ;
    mCompute = ComputeImpl.asInterface(computeBinder);
    try {
      System.out.println("3+5=" + mCompute.add(3, 5));
    } catch (RemoteException e) {
      e.printStackTrace();
    }
  }
}
```