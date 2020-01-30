title: AsyncTask Mechanism
date: 2019-2-24
tags: [Android]
categories: Android开发艺术探索
description: 　　
---
## 源码解析

### 构造方法
```java
public AsyncTask() {
        this((Looper) null);
    }
	public AsyncTask(@Nullable Looper callbackLooper) {
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()
            : new Handler(callbackLooper);
...
    }
```
由此可看出默认AsyncTask是在主线程运行的。且onPreExecute、onPostExecute也定义了@MainThread
### 业务逻辑
进入AsyncTask的execute（）
```java
    @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);//sDefaultExecutor is a sreial executor
    }
    @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }
```
sDefaultExecutor是一个串行的线程池，先执行onPreExecute().然后将params封装到mWorker(WorkerRunnable)中.而mWorker又是被封装到mFuture(FutureTask也是个Runnable)中的。然后sDefaultExecutor线程池开始执行FututreTask。进入sDefaultExecutor
```java
    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

  ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
public static final Executor SERIAL_EXECUTOR = new SerialExecutor()
private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;

   private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
execute()首先会将FutureTask对象插入到任务队列mTasks上，如果此时没有正在执行的任务，则调用scheduleNext()让THREAD_POOL_EXECUTOR执行任务。同时一个任务执行结束后，会自动地调度下一个任务开始执行。

AsyncTask中有两个线程池sDefaultExecutor（SerialExecutor:内部封装了ArrayDequeue）和 THREAD_POOL_EXECUTOR (Customized Executor:内部封装了BlockingQueue) 和一个InternalHandler(Handler).SerialExecutor用于任务的排队，而 THREAD_POOL_EXECUTOR 用于执行任务。IntenalHandler用于子线程切换主线程。进入mFurue的run方法。
```java
public void run() {
        if (state != NEW ||
            !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
```
其中callable即为封装的mWorker(WorkerRunnable).进入构造方法中定义WorkerRunnable的call方法
```java
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        }
```
mTaskInvoked设为true，表示任务已被调用过。再执行doInBackGround.将返回值传给postResult().进入postResult()

```java
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```
 postResult（）会通过sHandler发送一个MESSAGE_POST_RESULT的消息。进入sHandler
```java
    private static InternalHandler sHandler;
	
private static class InternalHandler extends Handler {
        public InternalHandler(Looper looper) {
            super(looper);
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
	private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
```
sHadler在收到MESSAGE_POST_RESULT执行AsyncTasl。finish()，而在finish()中，若AsyncTask是被取消执行，则调用onCancelled(result),否则调用onPostExecutor(result).将doInBackGround()的返回值传给了主线程。
## 总结
1. 先执行onPreExecute. 将传入params封装到mWorker中，而mWorker是封装到mFuture中的。
2. 将mFuture压入SerialExecutor任务队列，若队列空，让THREAD_POOL_EXECUTOR开始执行mFuture. 执行mFuture的run()会调用mWorker的call（）
3. 在call()调用doInBackGround()获得返回值，并将返回值传给sHandler
4. sHandler接到消息调用AsyncTask的finish()
5. 在finish()中调用onPostExecute()

## 小心翻车
- AsyncTask不必在主线程定义
- execute不必在主线程调用
- 不能自己直接调用onPre/onPost/doInBackGround/onProgressUpdated
- 一个AsyncTask对象只能执行一次（否则会抛出异常IllegalStateException）

之前在开发者艺术探索中读到不能在子线程中定义AsyncTask定义和使用execute。但是经过实践，发现从sdk22以后，是可以在子线程中进行定义和execute的。因为Handler默认是使用MainLooper，无论在子线程中是否定义Looper。
```java
    public AsyncTask() {
        this((Looper) null);//1
    }
	    public AsyncTask(@Nullable Looper callbackLooper) {
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()//2
            : new Handler(callbackLooper);
			....
			}
			
			    private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler(Looper.getMainLooper());//3
            }
            return sHandler;
        }
    }
```
1. 直接new AsyncTask会调用Looper为空的重载方法
2. Handler = getMainHandler
3. Looper = sMainLooper

测试代码如下

```java
 new Thread(new Runnable() {
            @Override public void run() {
                MyAsync myAsync = new MyAsync();
                myAsync.execute();
            }
        }).start();
		    public class MyAsync extends AsyncTask {
 @Override protected void onPreExecute() {
            Log.i("TAG", "onPreExecute: ");
            super.onPreExecute();
        }

        @Override protected Object doInBackground(Object[] objects) {
            Log.i("TAG", "doInBackground: ");
            return null;
        }
    }
```
当然子线程中使用AsyncTask不能在onPreExecute,onPostExecute中进行UI操作，因为AsyncTask的实例创建在子线程，onPreExecute,onPostExecute也都运行在子线程。

在