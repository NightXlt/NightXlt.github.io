title: Glide源码解析
date: 2019-3-11
tags: [Android]
categories: Android开发艺术探索
description: 加载中。。。
---
## 简介

``` java
Glide.with(this)
    .load(url)
    .into(imageView);
```

Glide加载执行流程
1. 创建request
2. 执行加载
3. 回调刷新UI

![glide_running](/images/glide_resolve.png)



## 创建request
### with()
从Glide.with()开始

``` java
public static RequestManager with(Activity activity) {
  return getRetriever(activity).get(activity);
}
public static RequestManager with(FragmentActivity activity) {
  return getRetriever(activity).get(activity);
}
public static RequestManager with(Fragment fragment) {
  return getRetriever(fragment.getActivity()).get(fragment);
}
public static RequestManager with(android.app.Fragment fragment) {
  return getRetriever(fragment.getActivity()).get(fragment);
}
 public static RequestManager with(Context context) {
  return getRetriever(context).get(context);
}
public static RequestManager with(View view) {
  return getRetriever(view.getContext()).get(view);
}
```
所有with()均返回了一个RequestManager,进入getRetriever（）
```java
private static RequestManagerRetriever getRetriever(@Nullable Context context) {
 ...
    return Glide.get(context).getRequestManagerRetriever();
  }
  
   /**
   * Get the singleton.
   *
   * @return the singleton
   */
  public static Glide get(Context context) {
    if (glide == null) {
      synchronized (Glide.class) {
        if (glide == null) {
          checkAndInitializeGlide(context);
        }
      }
    }
    return glide;
  }
```
Glide通过DCL创建单例，然后RequestManagerRetriever对象通过Glide单例获取。返回到上面 getRetriever().get()
```java
  @NonNull
  public RequestManager get(@NonNull Context context) {
    if (context == null) {
      throw new IllegalArgumentException("You cannot start a load on a null Context");
    } else if (Util.isOnMainThread() && !(context instanceof Application)) {
      if (context instanceof FragmentActivity) {
        return get((FragmentActivity) context);
      } else if (context instanceof Activity) {
        return get((Activity) context);
      } else if (context instanceof ContextWrapper) {
        return get(((ContextWrapper) context).getBaseContext());
      }
    }
    return getApplicationManager(context);
  }
  @NonNull
  public RequestManager get(@NonNull FragmentActivity activity) {
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    } else {
      assertNotDestroyed(activity);
      FragmentManager fm = activity.getSupportFragmentManager();
      return supportFragmentGet(
          activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
    }
  }

  @NonNull
  public RequestManager get(@NonNull Fragment fragment) {
    Preconditions.checkNotNull(fragment.getActivity(),
          "You cannot start a load on a fragment before it is attached or after it is destroyed");
    if (Util.isOnBackgroundThread()) {
      return get(fragment.getActivity().getApplicationContext());
    } else {
      FragmentManager fm = fragment.getChildFragmentManager();
      return supportFragmentGet(fragment.getActivity(), fm, fragment, fragment.isVisible());
    }
  }

  private RequestManager supportFragmentGet(Context context, FragmentManager fm,
      Fragment parentHint) {
      //1 获取managerFragment
    SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm, parentHint);
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
      // TODO(b/27524013): Factor out this Glide.get() call.
      Glide glide = Glide.get(context);
      //2 创建requestManager ，并传入了Lifecycle
      requestManager = factory.build(glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode());
     //3 缓存requestManager,保证一个Activity对应一个requestManager
      current.setRequestManager(requestManager);
    }
    return requestManager;
  }
  
  //4 创建并添加一个SupportRequestManagerFragment
  SupportRequestManagerFragment getSupportRequestManagerFragment(
      final FragmentManager fm, Fragment parentHint) {
    SupportRequestManagerFragment current =
        (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
      current = pendingSupportRequestManagerFragments.get(fm);
      if (current == null) {
        current = new SupportRequestManagerFragment();
        current.setParentFragmentHint(parentHint);
        pendingSupportRequestManagerFragments.put(fm, current);
        //5 这里添加一个隐藏的Fragment
        fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
        handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
      }
    }
    return current;
  }
}
```
get方法重载参数虽然很多，但最终就返回两种类型的requestManager。一种是ApplicationManager,它自动和应用的生命周期同步，应用退出，Glide也就停止加载；另外一种则是带有Fragment生命周期的requestManager。对应上述代码中注释，可以看出，Glide添加一个隐藏的Fragment，获取对应的生命周期回调事件，这样就可在Activity销毁时停止加载图片了。这种方式比较巧妙，不仅是Glide，RxPermission项目也是这样使用的。

小结：Glide.with(）：它主要完成了Glide实例化，并返回requestManager对象。

### load(url)

``` java
 public RequestBuilder<Drawable> load(@Nullable Object model) {
    return asDrawable().load(model);
  }
  //asDrawable().load(model);最终会调用至loadGeneric
  private RequestBuilder<TranscodeType> loadGeneric(@Nullable Object model) {
    this.model = model;
    isModelSet = true;
    return this;
  }
```
返回一个request构造器，接着进入into.

```java
  private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> options,
      Executor callbackExecutor) {
    Preconditions.checkNotNull(target);
   

    Request request = buildRequest(target, targetListener, options, callbackExecutor);//1. 构建request

    Request previous = target.getRequest();
    if (request.isEquivalentTo(previous)
        && !isSkipMemoryCacheWithCompletePreviousRequest(options, previous)) {
      request.recycle();
    ...
    requestManager.clear(target);
    target.setRequest(request);
	//2. 执行request（位于下一小节，执行加载）
    requestManager.track(target, request);

    return target;
  }

```
进入1处构建request
```java
private Request buildRequest(
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> requestOptions,
      Executor callbackExecutor) {
    return buildRequestRecursive(
        target,
        targetListener,
        /*parentCoordinator=*/ null,
        transitionOptions,
        requestOptions.getPriority(),
        requestOptions.getOverrideWidth(),
        requestOptions.getOverrideHeight(),
        requestOptions,
        callbackExecutor);
  }
  
  
  private Request buildRequestRecursive(
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      BaseRequestOptions<?> requestOptions,
      Executor callbackExecutor) {
...
    Request mainRequest =
        buildThumbnailRequestRecursive(
            target,
            targetListener,
            parentCoordinator,
            transitionOptions,
            priority,
            overrideWidth,
            overrideHeight,
            requestOptions,
            callbackExecutor);

    if (errorRequestCoordinator == null) {
      return mainRequest;
    }
  ...
  }
  
  private Request buildThumbnailRequestRecursive(
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      BaseRequestOptions<?> requestOptions,
      Executor callbackExecutor) {
 ...//默认无缩略图情况下
      // Base case: no thumbnail.
      return obtainRequest(
          target,
          targetListener,
          requestOptions,
          parentCoordinator,
          transitionOptions,
          priority,
          overrideWidth,
          overrideHeight,
          callbackExecutor);
   
  }
```
进入obtainRequest中

```java
  private Request obtainRequest(
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> requestOptions,
      RequestCoordinator requestCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      Executor callbackExecutor) {
    return SingleRequest.obtain(
        context,
        glideContext,
        model,
        transcodeClass,
        requestOptions,
        overrideWidth,
        overrideHeight,
        priority,
        target,
        targetListener,
        requestListeners,
        requestCoordinator,
        glideContext.getEngine(),
        transitionOptions.getTransitionFactory(),
        callbackExecutor);
  }

```
最终通过RequestBuilder生成了一个SingleRequest实例。这个SingleRequest类中有各种属性，大部分都是默认了，当然可以在使用时通过RequestOptions配置。
### 执行加载
```java
  synchronized void track(@NonNull Target<?> target, @NonNull Request request) {
    targetTracker.track(target);
    requestTracker.runRequest(request);
  }
    public void runRequest(Request request) {
    requests.add(request);
    if (!isPaused) {
      request.begin();
    } else {
      pendingRequests.add(request);
    }
  }
  
    //以下是SingleRequest类中begin方法实现。
  @Override
  public void begin() {
  ...
    // 1
    if (status == Status.COMPLETE) {
      onResourceReady(resource, DataSource.MEMORY_CACHE);
      return;
    }
    // 2
    status = Status.WAITING_FOR_SIZE;
    if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
      onSizeReady(overrideWidth, overrideHeight);
    } else {
      target.getSize(this);
    }
    // 3
    if ((status == Status.RUNNING || status == Status.WAITING_FOR_SIZE)
        && canNotifyStatusChanged()) {
      target.onLoadStarted(getPlaceholderDrawable());
    }
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
      logV("finished run method in " + LogTime.getElapsedMillis(startTime));
    }
  }
```
1. 加载完成回调。这里是加载、缩放、转换之后的数据，可直接用于UI显示。后面再分析是怎么回调刷新UI的。
2. 这里主要是判断overrideWidth, overrideHeight是否可用。分两种情况：1).如果设置了override(int width, int height) ,直接处理onSizeReady方法逻辑。2).没有设置override，Glide就会等到系统计算完组件宽高后再回调onSizeReady。所以两种情况最后都会调用onSizeReady方法。
3. 开始前，回调设置placeholderDrawable

长宽测量完毕进入onSizeReady()
```java
@Override
  public void onSizeReady(int width, int height) {
    ...省略
    //Engine类的load方法，正式步入加载流程。
    loadStatus = engine.load(
        glideContext,
        model,//对应myUrl,图片地址
        requestOptions.getSignature(),
        this.width,
        this.height,
        requestOptions.getResourceClass(),//默认是Object.class
        transcodeClass, //默认Drawbale.class
        priority,
        requestOptions.getDiskCacheStrategy(),//磁盘缓存策略。默认是DiskCacheStrategy.AUTOMATIC
        requestOptions.getTransformations(),
        requestOptions.isTransformationRequired(),
        requestOptions.isScaleOnlyOrNoTransform(),
        requestOptions.getOptions(),
        requestOptions.isMemoryCacheable(),
        requestOptions.getUseUnlimitedSourceGeneratorsPool(),
        requestOptions.getOnlyRetrieveFromCache(),
        this);
  }
```
姗姗来迟的load方法中包含了Glid的缓存策略思想
```java
  public synchronized <R> LoadStatus load(
      GlideContext glideContext,
      Object model,
      Key signature,
      int width,
      int height,
      Class<?> resourceClass,
      Class<R> transcodeClass,
      Priority priority,
      DiskCacheStrategy diskCacheStrategy,
      Map<Class<?>, Transformation<?>> transformations,
      boolean isTransformationRequired,
      boolean isScaleOnlyOrNoTransform,
      Options options,
      boolean isMemoryCacheable,
      boolean useUnlimitedSourceExecutorPool,
      boolean useAnimationPool,
      boolean onlyRetrieveFromCache,
      ResourceCallback cb,
      Executor callbackExecutor) {
    long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;
// 生成缓存的key
    EngineKey key = keyFactory.buildKey(model, signature, width, height, transformations,
        resourceClass, transcodeClass, options);
// 先从活动资源 (Active Resources) - 正在显示的资源去取
    EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
    if (active != null) {
      cb.onResourceReady(active, DataSource.MEMORY_CACHE);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from active resources", startTime, key);
      }
      return null;
    }
// 再从内存缓存 (Memory cache) - 显示过的资源去取
    EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
    if (cached != null) {
      cb.onResourceReady(cached, DataSource.MEMORY_CACHE);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from cache", startTime, key);
      }
      return null;
    }
	////检查当前Request是否正在执行。
    EngineJob<?> current = jobs.get(key, onlyRetrieveFromCache);
    if (current != null) {
      current.addCallback(cb, callbackExecutor);
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Added to existing load", startTime, key);
      }
      return new LoadStatus(cb, current);
    }
// 创建后台任务
    EngineJob<R> engineJob =
        engineJobFactory.build(
            key,
            isMemoryCacheable,
            useUnlimitedSourceExecutorPool,
            useAnimationPool,
            onlyRetrieveFromCache);

    DecodeJob<R> decodeJob =
        decodeJobFactory.build(
            glideContext,
            model,
            key,
            signature,
            width,
            height,
            resourceClass,
            transcodeClass,
            priority,
            diskCacheStrategy,
            transformations,
            isTransformationRequired,
            isScaleOnlyOrNoTransform,
            onlyRetrieveFromCache,
            options,
            engineJob);

    jobs.put(key, engineJob);

    engineJob.addCallback(cb, callbackExecutor);
	//开启任务
    engineJob.start(decodeJob);

    if (VERBOSE_IS_LOGGABLE) {
      logWithTimeAndKey("Started new load", startTime, key);
    }
    return new LoadStatus(cb, engineJob);
  }

```
- 内存：
活动资源 (Active Resources) - 正在显示的资源,保护显示的图片不会被LruCache算法回收掉。
内存缓存 (Memory cache) - 显示过的资源，采取LRU算法置换
- 磁盘：
资源类型（Resource） - 被解码、转换后的资源
数据来源 (Data) - 源文件（未处理过）

进入磁盘缓存的start()
```java
//start方法就是根据diskCacheStrategy策略获取一个executor来执行DecodeJob
public void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    //这里根据缓存策略，决定使用哪个Executor。默认情况返回diskCacheExecutor。
    //共三种执行器：diskCacheExecutor、sourceExecutor、sourceUnlimitedExecutor对应文章前面给出的流程图。
    GlideExecutor executor = decodeJob.willDecodeFromCache()
        ? diskCacheExecutor
        : getActiveSourceExecutor();
    executor.execute(decodeJob);
  }
//DecodeJob实现了Runnable接口。直接来看它的run方法。
 @Override
  public void run() {
    TraceCompat.beginSection("DecodeJob#run");
    try {
      if (isCancelled) {
        notifyFailed();
        return;
      }
      runWrapped();//看这里！
    } catch (RuntimeException e) {
      if (Log.isLoggable(TAG, Log.DEBUG)) {
        Log.d(TAG, "DecodeJob threw unexpectedly"
            + ", isCancelled: " + isCancelled
            + ", stage: " + stage, e);
      }
     ....
  }
//接着看runWrapped方法。
//RunReason是一个枚举，默认值为INITIALIZE。区分任务目的。
private void runWrapped() {
     switch (runReason) {
      case INITIALIZE:
        stage = getNextStage(Stage.INITIALIZE);
        currentGenerator = getNextGenerator();
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
  }
  
  //获取任务执行阶段：初始化、读取转换后的缓存、读取原文件缓存、原文件加载、结束状态。
  private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        // Skip loading from source if the user opted to only retrieve the resource from cache.
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
      default:
        throw new IllegalArgumentException("Unrecognized stage: " + current);
    }
  }
//根据上一个方法确定的stage，创建对应的Generator(可把它简单理解成资源加载器）
  private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
      case RESOURCE_CACHE:
        //从转换后的缓存中读取文件
        return new ResourceCacheGenerator(decodeHelper, this);
      case DATA_CACHE:
       //从原文件缓存中读取文件
        return new DataCacheGenerator(decodeHelper, this);
      case SOURCE:
       //没有缓存，重新加载资源（比如：网络图片、本地文件）
        return new SourceGenerator(decodeHelper, this);
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized stage: " + stage);
    }
  }
 //这里开始加载执行
  private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    //这里Generator.startNext()方法中就是加载过程，如果成功加载则返回true并跳出循环，否则切换Generator继续执行。
    while (!isCancelled && currentGenerator != null
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();
     //如果任务执行到去加载资源（也就是没有命中磁盘缓存），且切换任务执行环境
      if (stage == Stage.SOURCE) {
        reschedule();
        return;
      }
    }
    // We've run out of stages and generators, give up.
    if ((stage == Stage.FINISHED || isCancelled) && !isStarted) {
      notifyFailed();
    }
    // Otherwise a generator started a new load and we expect to be called back in
    // onDataFetcherReady.
  }
 
  @Override
  public void reschedule() {
    //更改执行目标为：SOURCE服务。当然也只有在stage == Stage.SOURCE的情况下会被调用。
    runReason = RunReason.SWITCH_TO_SOURCE_SERVICE;
    callback.reschedule(this);//这里callback正是EngineJob。
  } 
  
  //代码跟进EngineJob类中，可以看到实现方法。
  @Override
  public void reschedule(DecodeJob<?> job) {
    // 可以看到，这里获取的SourceExecutor来执行decodeJob。
    //也就巧妙地将此decodeJob任务从cacheExecutor切换到了SourceExecutor，这样分工协作更加高效。
    getActiveSourceExecutor().execute(job);
  }
```
从最开始没有命中内存缓存开始，然后执行Engine的start方法
1. 默认情况会获取到cacheExecutor执行器来执行decodeJob任务；
2. 继续decodeJob的run方法，因为RunReason==INITIALIZE，接着获取stage，默认会返回Stage.RESOURCE_CACHE,这时通过getNextGenerator就返回了ResourceCacheGenerator加载
3. 紧接着就是调用 ResourceCacheGenerator的startNext方法 ，从转换后的缓存中读取已缓存的资源，如果命中则结束任务并回调结果，反之，任务切换到DataCacheGenerator加载器继续执行
4. 若还是未命中，则切换到SourceGenerator加载器（第一次加载，由于没有任何缓存，就会走到这里），这时会通过任务调度，将线程运行环境切换到 SourceExecutor执行器来执行，最后，待SourceGenerator加载完成后结束任务，回调结果，流程结束。

网络加载过程
Glide底层是采用HttpUrlConnection进行网络请求，也可更换为OkHttp请求连接。
### 回调刷新UI
回到DecodeJob类中
```java
@Override
  public void onDataFetcherReady(Key sourceKey, Object data, DataFetcher<?> fetcher,
      DataSource dataSource, Key attemptedKey) {
      ...
      //资源加载完成后回调，执行
      decodeFromRetrievedData();
      ...
  }
  //处理源数据
  private void decodeFromRetrievedData() {
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
      logWithTimeAndKey("Retrieved data", startFetchTime,
          "data: " + currentData
          + ", cache key: " + currentSourceKey
          + ", fetcher: " + currentFetcher);
    }
    Resource<R> resource = null;
    try {
      //比如缩放，转换等。会判断currentDataSource类型。
      resource = decodeFromData(currentFetcher, currentData, currentDataSource);
    } catch (GlideException e) {
      e.setLoggingDetails(currentAttemptingKey, currentDataSource);
      exceptions.add(e);
    }
    if (resource != null) {
      notifyEncodeAndRelease(resource, currentDataSource);
    } else {
      runGenerators();
    }
  }
  private void notifyEncodeAndRelease(Resource<R> resource, DataSource dataSource) {
    if (resource instanceof Initializable) {
      ((Initializable) resource).initialize();
    }
    Resource<R> result = resource;
    LockedResource<R> lockedResource = null;
    if (deferredEncodeManager.hasResourceToEncode()) {
      lockedResource = LockedResource.obtain(resource);
      result = lockedResource;
    }
    //通知完成，并回调。
    notifyComplete(result, dataSource);
    stage = Stage.ENCODE;
    try {
      //磁盘缓存
      if (deferredEncodeManager.hasResourceToEncode()) {
        deferredEncodeManager.encode(diskCacheProvider, options);
      }
    } finally {
      if (lockedResource != null) {
        lockedResource.unlock();
      }
      onEncodeComplete();
    }
  }
  
  private void notifyComplete(Resource<R> resource, DataSource dataSource) {
    setNotifiedOrThrow();
    //这里的callback就是EngineJob
    callback.onResourceReady(resource, dataSource);
  }
```
DecodeJob加载完数据后，会做转换、磁盘缓存等操作。进入到EngineJob的onResourceReady
```java
@Override
  public void onResourceReady(Resource<R> resource, DataSource dataSource) {
    this.resource = resource;
    this.dataSource = dataSource;
    //将回调过程通过Handler切换到主线程
    MAIN_THREAD_HANDLER.obtainMessage(MSG_COMPLETE, this).sendToTarget();
  }
  
   @Override
    public boolean handleMessage(Message message) {
      EngineJob<?> job = (EngineJob<?>) message.obj;
      switch (message.what) {
        case MSG_COMPLETE:
          job.handleResultOnMainThread();
          break;
        case MSG_EXCEPTION:
          job.handleExceptionOnMainThread();
          break;
        case MSG_CANCELLED:
          job.handleCancelledOnMainThread();
          break;
        default:
          throw new IllegalStateException("Unrecognized message: " + message.what);
      }
      return true;
    }
  @Synthetic
  void handleResultOnMainThread() {
    stateVerifier.throwIfRecycled();
    if (isCancelled) {
      resource.recycle();
      release(false /*isRemovedFromQueue*/);
      return;
    } else if (cbs.isEmpty()) {
      throw new IllegalStateException("Received a resource without any callbacks to notify");
    } else if (hasResource) {
      throw new IllegalStateException("Already have resource");
    }
    engineResource = engineResourceFactory.build(resource, isCacheable);
    hasResource = true;
    // Hold on to resource for duration of request so we don't recycle it in the middle of
    // notifying if it synchronously released by one of the callbacks.
    engineResource.acquire();
    //通知并缓存到activeResources（内存缓存中的一种）中。
    listener.onEngineJobComplete(key, engineResource);
    for (ResourceCallback cb : cbs) {
      if (!isInIgnoredCallbacks(cb)) {
        engineResource.acquire();
        //这里将回调到SingleRequest的onResourceReady中。
        cb.onResourceReady(engineResource, dataSource);
      }
    }
    // Our request is complete, so we can release the resource.
    engineResource.release();
    release(false /*isRemovedFromQueue*/);
  }
  
  //继续跟进SingleRequest类
  @Override
  public void onResourceReady(Resource<?> resource, DataSource dataSource) {
    ...
    onResourceReady((Resource<R>) resource, (R) received, dataSource);
  }
  private void onResourceReady(Resource<R> resource, R result, DataSource dataSource) {
    // We must call isFirstReadyResource before setting status.
    boolean isFirstResource = isFirstReadyResource();
    status = Status.COMPLETE;
    this.resource = resource;
   
    if (requestListener == null
        || !requestListener.onResourceReady(result, model, target, dataSource, isFirstResource)) {
      Transition<? super R> animation =
          animationFactory.build(dataSource, isFirstResource);
    //如果没有设置requestListener或者未消耗事件，就会回调target的onResourceReady方法。
    //默认的target是DrawableImageViewTarget     
      target.onResourceReady(result, animation);
    }
    notifyLoadSuccess();
  }
```
回调从Decodejob出来，在EngineJob中切换到主线程并一路回调到DrawableImageViewTarget中，至于为什么默认是DrawableImageViewTarget，请查看RequestBuilder中into方法。下面我们再看下DrawableImageViewTarget相关代码，也是设置显示图片的地方。
```java
ublic class DrawableImageViewTarget extends ImageViewTarget<Drawable> {
  public DrawableImageViewTarget(ImageView view) {
    super(view);
  }
  @Override
  protected void setResource(@Nullable Drawable resource) {
    //实现了该方法。简单将drawable设置给imageview。
    view.setImageDrawable(resource);
  }
}
public abstract class ImageViewTarget<Z> extends ViewTarget<ImageView, Z>
    implements Transition.ViewAdapter {
  public ImageViewTarget(ImageView view) {
    super(view);
  }
  @Override
  public void onResourceReady(Z resource, @Nullable Transition<? super Z> transition) {
    if (transition == null || !transition.transition(resource, this)) {
      setResourceInternal(resource);
    } else {
      maybeUpdateAnimatable(resource);
    }
  }
  private void setResourceInternal(@Nullable Z resource) {
    maybeUpdateAnimatable(resource);
    setResource(resource);
  }
  private void maybeUpdateAnimatable(@Nullable Z resource) {
    if (resource instanceof Animatable) {
      animatable = (Animatable) resource;
      animatable.start();
    } else {
      animatable = null;
    }
  }
  protected abstract void setResource(@Nullable Z resource);
}
```
onResourceReady在父类ImageViewTarget中回调，然后调用setResource将图片设置并显示出来。代码执行到这里，回调过程也就结束了。