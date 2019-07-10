title: ContentProvider源码解析
date: 2019-3-9
tags: [Android,源码解析]
categories: Android开发艺术探索
description: 　　
---
## 基础知识
1. `当ContentProvider所在进程启动时，ContentProvider的onCreate会先于Application的onCreate()执行，并且发布到AMS中。`
2. ContentProvider是个抽象类，必须继承ContentProvider重写方法方可使用
3. 外界无法直接访问ContentProvider，必须通过ContentResolver来获取AMS中的CotentProvider的Binder接口IContentProvider
4. ContentResolver是一个抽象类，Context.getContentResolver()获取的实际是实现了ContentResolver接口的ApplicationContentResolver对象
5. 当ContentProvider没有启动，其他应用进程访问ContentProvider时，ContentProvider创建的同时ContentProvider所在进程会启动
## ContentResolver调用
以query为例，分析ContentProvider源码
先从ContentResolver开始哈,毕竟它是发起访问的。
```java    public final @Nullable Cursor query(final @RequiresPermission.Read @NonNull Uri uri,
            @Nullable String[] projection, @Nullable Bundle queryArgs,
            @Nullable CancellationSignal cancellationSignal) {
        Preconditions.checkNotNull(uri, "uri");
        IContentProvider unstableProvider = acquireUnstableProvider(uri);
...
    }
```
首先获取IContentProvider对象，acquireUnstableProvider最终调用acquireProvider（抽象方法），而ContentResolver的实现类是源码ContexImpl下的ApplicationContentResolver.（要从AndoridXref看）
```java
 private static final class ApplicationContentResolver extends ContentResolver {
       public ApplicationContentResolver(Context context, ActivityThread mainThread) {
          super(context);
          mMainThread = mainThread;
      }

      @Override
      protected IContentProvider acquireProvider(Context context, String name) {
          return mMainThread.acquireProvider(context, name);
      }

      @Override
      protected IContentProvider acquireExistingProvider(Context context, String name) {
           return mMainThread.acquireExistingProvider(context, name);
       }

       @Override
       public boolean releaseProvider(IContentProvider provider) {
           return mMainThread.releaseProvider(provider);
       }

       private final ActivityThread mMainThread;
   }
```
ApplicationContentResolver的acquireProvider调用了ActivityThread的acquireProvider（）
```java
    public final IContentProvider acquireProvider(
            Context c, String auth, int userId, boolean stable) {
        final IContentProvider provider = acquireExistingProvider(c, auth, userId, stable);//1
        if (provider != null) {
            return provider;
        }

        // There is a possible race here.  Another thread may try to acquire
        // the same provider at the same time.  When this happens, we want to ensure
        // that the first one wins.
        // Note that we cannot hold the lock while acquiring and installing the
        // provider since it might take a long time to run and it could also potentially
        // be re-entrant in the case where the provider is in the same process.
        ContentProviderHolder holder = null;//2
        try {
            synchronized (getGetProviderLock(auth, userId)) {
                holder = ActivityManager.getService().getContentProvider(
                        getApplicationThread(), auth, userId, stable);
            }
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
        if (holder == null) {
            Slog.e(TAG, "Failed to find provider info for " + auth);
            return null;
        }

        // Install provider will increment the reference count for us, and break
        // any ties in the race.
        holder = installProvider(c, holder, holder.info,
                true /*noisy*/, holder.noReleaseNeeded, stable);
        return holder.provider;
    }

```
1处：先从ActivityThread中的 ArrayMap<ProviderKey, ProviderClientRecord> mProviderMap查找是否存在目标ContentProvider,如果存在就直接返回。

2处：如果目前ContentProvider没有启动，就会开启RPC远程调用给AMS让其启动目标ContentProvider，最后通过installProvider修改引用计数。而ContentProvider的启动伴随着App进程的启动。

启动流程为先启动ContentProvider所在进程，然后再启动ContentProvider。然后再启动ContentProvider.由于一个APP从启动到主页面显示经历了哪些过程？已经做过笔记，不再赘述。直接进入handleBindApplication中
## ContentProvider创建
在handleBindApplication中ContentProvider的创建分为四个步骤。

### 创建ContextImpl 和 Instrumentation
```java
            final ContextImpl instrContext = ContextImpl.createAppContext(this, pi);

            try {
                final ClassLoader cl = instrContext.getClassLoader();
                mInstrumentation = (Instrumentation)
                    cl.loadClass(data.instrumentationName.getClassName()).newInstance();
            } catch (Exception e) {
                throw new RuntimeException(
                    "Unable to instantiate instrumentation "
                    + data.instrumentationName + ": " + e.toString(), e);
            }

            final ComponentName component = new ComponentName(ii.packageName, ii.name);
            mInstrumentation.init(this, instrContext, appContext, component,
                    data.instrumentationWatcher, data.instrumentationUiAutomationConnection);

```
创建ContextImpl，再通过反射创建Instrumentation，并初始化Instrumentation

### 创建Application对象（并未调用其OnCreate方法）
```java
 app = data.info.makeApplication(data.restrictedBackupMode, null);

            // Propagate autofill compat state
            app.setAutofillCompatibilityEnabled(data.autofillCompatibilityEnabled);

            mInitialApplication = app;

```

### 启动ContentProvider，并调用其onCreate方法
```java
            if (!data.restrictedBackupMode) {
                if (!ArrayUtils.isEmpty(data.providers)) {
                    installContentProviders(app, data.providers);
                    // For process that contains content providers, we want to
                    // ensure that the JIT is enabled "at some point".
                    mH.sendEmptyMessageDelayed(H.ENABLE_JIT, 10*1000);
                }
            }
private void installContentProviders(
            Context context, List<ProviderInfo> providers) {
        final ArrayList<ContentProviderHolder> results = new ArrayList<>();

        for (ProviderInfo cpi : providers) {
            if (DEBUG_PROVIDER) {
                StringBuilder buf = new StringBuilder(128);
                buf.append("Pub ");
                buf.append(cpi.authority);
                buf.append(": ");
                buf.append(cpi.name);
                Log.i(TAG, buf.toString());
            }
            ContentProviderHolder cph = installProvider(context, null, cpi,
                    false /*noisy*/, true /*noReleaseNeeded*/, true /*stable*/);
            if (cph != null) {
                cph.noReleaseNeeded = true;
                results.add(cph);
            }
        }

        try {
            ActivityManager.getService().publishContentProviders(
                getApplicationThread(), results);
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
    }
```
调用installContentProviders会遍历ProviderInfo的list，一一调用installProvider()来启动它们。来启动ContentProvider.接着将已启动的ContentProvider发布到AMS中，AMS会把它们存储到ProvideMap中，供ContentResolver查询
进入installProvider
```java
    private ContentProviderHolder installProvider(Context context,
            ContentProviderHolder holder, ProviderInfo info,
            boolean noisy, boolean noReleaseNeeded, boolean stable) {
        ContentProvider localProvider = null;
        IContentProvider provider;
    .....
                if (DEBUG_PROVIDER) Slog.v(
                    TAG, "Instantiating local provider " + info.name);
                // XXX Need to create the correct context for this provider.
                localProvider.attachInfo(c, info);
            } catch (java.lang.Exception e) {
                if (!mInstrumentation.onException(null, e)) {
                    throw new RuntimeException(
                            "Unable to get provider " + info.name
                            + ": " + e.toString(), e);
                }
                return null;
            }
        }
		.....
		}

```
进入attachInfo
```java
private void attachInfo(Context context, ProviderInfo info, boolean testing) {
        mNoPerms = testing;

        /*
         * Only allow it to be set once, so after the content service gives
         * this to us clients can't change it.
         */
        if (mContext == null) {
            mContext = context;
            if (context != null) {
                mTransport.mAppOpsManager = (AppOpsManager) context.getSystemService(
                        Context.APP_OPS_SERVICE);
            }
            mMyUid = Process.myUid();
            if (info != null) {
                setReadPermission(info.readPermission);
                setWritePermission(info.writePermission);
                setPathPermissions(info.pathPermissions);
                mExported = info.exported;
                mSingleUser = (info.flags & ProviderInfo.FLAG_SINGLE_USER) != 0;
                setAuthorities(info.authority);
            }
            ContentProvider.this.onCreate();
        }
    }
```
在attachInfo中调用了ContentProvider的onCreate方法

### 调用Application的onCreate方法
在handleBindApplication中启动完Binder后调用Application的onCreate方法
```java
                mInstrumentation.callApplicationOnCreate(app);
```


## 访问ContentProvider
经过上述步骤，ContentProvider已经成功启动，且其所在进程的Application也已经启动起来，所以访问进程的ContentResolver就可以通过AMS访问这个ContentProvider.
回到ContentResolver中的query方法
```java
    public final @Nullable Cursor query(final @RequiresPermission.Read @NonNull Uri uri,
            @Nullable String[] projection, @Nullable Bundle queryArgs,
            @Nullable CancellationSignal cancellationSignal) {
        Preconditions.checkNotNull(uri, "uri");
        IContentProvider unstableProvider = acquireUnstableProvider(uri);
        if (unstableProvider == null) {
            return null;
        }
        IContentProvider stableProvider = null;
        Cursor qCursor = null;
        try {
            long startTime = SystemClock.uptimeMillis();

            ICancellationSignal remoteCancellationSignal = null;
            if (cancellationSignal != null) {
                cancellationSignal.throwIfCanceled();
                remoteCancellationSignal = unstableProvider.createCancellationSignal();
                cancellationSignal.setRemote(remoteCancellationSignal);
            }
            try {
                qCursor = unstableProvider.query(mPackageName, uri, projection,
                        queryArgs, remoteCancellationSignal);
            } catch (DeadObjectException e) {
                // The remote process has died...  but we only hold an unstable
                // reference though, so we might recover!!!  Let's try!!!!
                // This is exciting!!1!!1!!!!1
                unstableProviderDied(unstableProvider);
                stableProvider = acquireProvider(uri);
                if (stableProvider == null) {
                    return null;
                }
                qCursor = stableProvider.query(
                        mPackageName, uri, projection, queryArgs, remoteCancellationSignal);
            }
       ...
    }

```
通过acquireUnstableProvider返回的是一个IContentProvider。以I开头的。没错，哼，又是你，Binder机制。IContentProvider就是Binder接口，那它调用query，岂不是代理在RPC咯，那找找服务端实现类和代理类。
```java
abstract public class ContentProviderNative extends Binder implements IContentProvider {
    public ContentProviderNative()
    {
        attachInterface(this, descriptor);
    }

    /**
     * Cast a Binder object into a content resolver interface, generating
     * a proxy if needed.
     */
    static public IContentProvider asInterface(IBinder obj)
    {
        if (obj == null) {
            return null;
        }
        IContentProvider in =
            (IContentProvider)obj.queryLocalInterface(descriptor);
        if (in != null) {
            return in;
        }

        return new ContentProviderProxy(obj);
    }
	.....
	}
	
	    class Transport extends ContentProviderNative {
        AppOpsManager mAppOpsManager = null;
        int mReadOp = AppOpsManager.OP_NONE;
        int mWriteOp = AppOpsManager.OP_NONE;

        ContentProvider getContentProvider() {
            return ContentProvider.this;
        }

        @Override
        public String getProviderName() {
            return getContentProvider().getClass().getName();
        }

        @Override
        public Cursor query(String callingPkg, Uri uri, @Nullable String[] projection,
                @Nullable Bundle queryArgs, @Nullable ICancellationSignal cancellationSignal) {
            validateIncomingUri(uri);
            uri = maybeGetUriWithoutUserId(uri);
            if (enforceReadPermission(callingPkg, uri, null) != AppOpsManager.MODE_ALLOWED) {
                // The caller has no access to the data, so return an empty cursor with
                // the columns in the requested order. The caller may ask for an invalid
                // column and we would not catch that but this is not a problem in practice.
                // We do not call ContentProvider#query with a modified where clause since
                // the implementation is not guaranteed to be backed by a SQL database, hence
                // it may not handle properly the tautology where clause we would have created.
                if (projection != null) {
                    return new MatrixCursor(projection, 0);
                }

                // Null projection means all columns but we have no idea which they are.
                // However, the caller may be expecting to access them my index. Hence,
                // we have to execute the query as if allowed to get a cursor with the
                // columns. We then use the column names to return an empty cursor.
                Cursor cursor = ContentProvider.this.query(
                        uri, projection, queryArgs,
                        CancellationSignal.fromTransport(cancellationSignal));
                if (cursor == null) {
                    return null;
                }

                // Return an empty cursor for all columns.
                return new MatrixCursor(cursor.getColumnNames(), 0);
            }
            final String original = setCallingPackage(callingPkg);
            try {
                return ContentProvider.this.query(
                        uri, projection, queryArgs,
                        CancellationSignal.fromTransport(cancellationSignal));
            } finally {
                setCallingPackage(original);
            }
        }

    ...
    }

```
ContentProviderNative继承Binder实现了ICotentProvider，而  ContentProvider.Transport 继承了ContentProviderNative。请注意哈，这里有一个神奇的地方，自己查了很久的书和网站得出结论。ContentProviderNative是一个android自己实现的contentProvider的代理类，然而ContentProvider.Transport继承了 ContentProviderNative 却是服务端实现接口。简言之，`当我们从ContentResolver中调用query时，通过 ContentProviderNative 作为代理 访问到了 其子类Transport中的query方法。`
在transport中，重写了父类的CRUD等方法。在transport中会调用当前ContentProvider的query方法。然后返回给代理，代理又返回给客户端。这样客户端就拿到了数据。
emmmmm，可以吃饭去了。