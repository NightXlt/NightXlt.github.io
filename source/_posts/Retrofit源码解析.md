title: Retrofit源码解析
date: 2019-3-3
tags: [Android,开源框架]
categories: 源码解析
description: 　　
---
## 简述Retrofit流程
![enter description here](/images/retrofit_process.png)
1. 通过注解配置API参数
2. 动态代理生成serviceMethod用以解析接口，生成request，返回Call对象
3. CallAdapter将Call转化为ExecutorCallbackcall
3. Converter(解析数据并转换成T)
## Retrofit的创建
```java
public interface Api {
    @GET("/api/{user} ")
    Call<Author> getAuthor(@Path("user") String user)
	//Rxjava
	@GET("/api/{user} ")
    Observable<Author> getAuthor(@Path("user") String user)
}

Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(API_URL)
    .addConverterFactory(GsonConverterFactory.create())
	.addCallAdapterFactory(RxJavaCallAdapterFactory.create())
    .build();
```
采用了建造者模式和外观模式；
外观模式：Retrofit给我们暴露的方法和类不多。核心类就是Retrofit，我们只管配置Retrofit，然后做请求。剩下的事情就跟上层无关了，只需要等待回调。这样大大降低了系统的耦合度。
建造者模式：动态添加对象属性，提高代码可读性。
进入Build（）
```java
public Builder() {
      this(Platform.get());//1
    }

```
Platform.get()：根据不同运行平台获取对应线程池
进入build()
```java
    public Retrofit build() {
      if (baseUrl == null) {//必须指定URL
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;//this.callFactory是构建时调用callFactory()传入设置的
      if (callFactory == null) {
        callFactory = new OkHttpClient();//如果没设置callFactory（），自动创建
      }

      Executor callbackExecutor = this.callbackExecutor;//callbackExecutor将回调传递到UI线程。
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);//adapterFactories存储对Call进行转换的工具如RxJavaCallAdapter,AndroidCallAdapter.
      adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));//获取默认的callBackAdapterFactory

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);// converterFactories 用于存储转化数据对象的工具如Gson

      return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
          callbackExecutor, validateEagerly);
    }
	
	
    public Builder callFactory(okhttp3.Call.Factory factory) {
      this.callFactory = checkNotNull(factory, "factory == null");
      return this;
    }
	////获取默认的callBackAdapter
static class Android extends Platform {
    @Override public Executor defaultCallbackExecutor() {
      return new MainThreadExecutor();
    }
 
    @Override CallAdapter.Factory defaultCallAdapterFactory(Executor callbackExecutor) {
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }
 
    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());
 
      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
```
  根据传入参数初始化成员变量，值得注意的是获取默认callBackExecutor。其中包含了Handler，用于默认的线程回调。所以onResponse回调在主线程，而okHttp并没有Handler。
## Call的创建过程
```java
Api api = retrofit.create(Api.class);
Call<ZhuanLanAuthor> call = api.getAuthor("night");
// 请求数据，并且处理response
call.enqueue(new Callback<Author>() {
    @Override
    public void onResponse(Response<Author> author) {
        System.out.println("name： " + author.getName());
    }
    @Override
    public void onFailure(Throwable t) {
    }
});

api.getAuthor("night") 
			.subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new Subscriber<Author>() {
                @Override
                public void onCompleted() {
                    Toast.makeText(MainActivity.this, "Get Top Movie Completed", Toast.LENGTH_SHORT).show();
                }

                @Override
                public void onError(Throwable e) {
				
                }

                @Override
                public void onNext(Author author) {
                   //do something
                }
            });
```
进入create方法
```java
@SuppressWarnings("unchecked") // Single-interface proxy creation guarded by parameter safety.
  public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);//1 ，注意哈
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);//2.
          }
        });
  }
  
  
  ServiceMethod<?, ?> loadServiceMethod(Method method) {
    ServiceMethod<?, ?> result = serviceMethodCache.get(method);//查询传入的方法是否有缓存。有的话就用缓存ServiceMethod；如果没有就创建一个。这是Retrofit的缓存方法机制
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder<>(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```
create()返回了一个Proxy.newProxyInstance动态代理对象，那么问题来了...动态代理是个什么东西？看Retrofit代码之前我知道Java动态代理是一个很重要的东西，比如在Spring框架里大量的用到，但是它有什么用呢？Java动态代理就是给了程序员一种可能：当你要调用某个Class的方法前或后，插入你想要执行的代码。比如你要执行某个操作前，你必须要判断这个用户是否登录，或者你在付款前，你需要判断这个人的账户中存在这么多钱。
### 为何使用动态代理
获取数据的代码就是这句:
```java
Call<Author> call = api.getAuthor("night");
```
上面`api`对象其实是一个动态代理对象，并不是一个真正的`Api`接口的implements产生的对象，当`api`对象调用`getAuthor`方法时会被动态代理拦截，然后调用`Proxy.newProxyInstance`方法中的InvocationHandler对象，它的`invoke`方法会传入3个参数：
- Object proxy: 代理对象，不关心这个
- Method method：调用的方法，就是`getAuthor`方法
- Object... args：方法的参数，就是`"night"`

而Retrofit关心的就是method和它的参数args，接下去Retrofit就会用Java反射获取到getAuthor方法的注解信息，配合args参数，创建一个`ServiceMethod`对象.

`ServiceMethod`就像是一个中央处理器，传入`Retrofit`对象和`Method`对象，调用各个接口和解析器，最终生成一个`Request`，包含api 的域名、path、http请求方法、请求头、是否有body、是否是multipart等等。最后返回一个`Call`对象，Retrofit2中Call接口的默认实现是OkHttpCall，它默认使用OkHttp3作为底层http请求client

使用Java动态代理的目的就要拦截被调用的Java方法，然后解析这个Java方法的注解，最后生成Request由OkHttp发送. 
### ServiceMethod
```java
public ServiceMethod build(){
      callAdapter = createCallAdapter();
      responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      responseConverter = createResponseConverter();

      for (Annotation annotation : methodAnnotations) {
        parseMethodAnnotation(annotation);
      }
...
int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        Type parameterType = parameterTypes[p];
        if (Utils.hasUnresolvableType(parameterType)) {
          throw parameterError(p, "Parameter type must not include a type variable or wildcard: %s",
              parameterType);
        }

        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
        if (parameterAnnotations == null) {
          throw parameterError(p, "No Retrofit annotation found.");
        }

        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }
  ...
      return new ServiceMethod<>(this);
    }
```
重点关注四个成员：callFactory，callAdapter，responseConverter和 parameterHandlers。

1. `callFactory` 负责创建 HTTP 请求，HTTP 请求被抽象为了 okhttp3.Call 类，它表示一个已经准备好，可以随时执行的 HTTP 请求；

2. `callAdapter` 把 retrofit2.Call< T > 转为 T（注意和 okhttp3.Call 区分开来，retrofit2.Call< T > 表示的是对一个 Retrofit 方法的调用），这个过程会发送一个 HTTP 请求，拿到服务器返回的数据（通过 okhttp3.Call 实现），并把数据转换为声明的 T 类型对象（通过 Converter<F, T> 实现）；可设置为 RxJavaCallAdapter
3. `responseConverter` 是 Converter< ResponseBody, T > 类型，负责把服务器返回的数据（JSON、XML、二进制或者其他格式，由 ResponseBody 封装）转化为 T 类型的对象；
4. `parameterHandlers` 则负责解析 API 定义时每个方法的参数，并在构造 HTTP 请求时设置参数；

<a name = "GsonConverterFactory声明处" href="#GsonConverterFactory调用处">GsonConverterFactory声明处</a>，接着回到create中的注释2处
### ExecutorCallbackCall
适配器模式： serviceMethod.callAdapter.adapt(okHttpCall);其中adapter方法会创建ExecutorCallbackCall.。而`ExecutorCallbackCall` 会将call回调主线程。如果采用的是RxJavaCallAdapter，则不会通过ExecutorCallbackCall进行发送网络请求以及回调主线程。

将传入的 okHttpCall 转换为 ExecutorCallbackCall， 为什么要转换呢？因为要给okHttpCall添加回调主线程功能。普通的OkHttp的Response都是运行在子线程必须要自己设置Handler回调,而ExecutorCallbackCall内置有Handler进行回调，见PlatForm处

```java
  @Override
  public CallAdapter<?, ?> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
    if (getRawType(returnType) != Call.class) {
      return null;
    }
    final Type responseType = Utils.getCallResponseType(returnType);
    return new CallAdapter<Object, Call<?>>() {
      @Override public Type responseType() {
        return responseType;
      }

      @Override public Call<Object> adapt(Call<Object> call) {
        return new ExecutorCallbackCall<>(callbackExecutor, call);
      }
    };
  }

```
进入ExecutorCallbackCall
```java
static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override public void enqueue(final Callback<T> callback) {
      if (callback == null) throw new NullPointerException("callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }
...
```
装饰者模式：ExecutorCallbackCall 主要是通过前面提及的 `callbackExecutor `将请求回调到主线程，当我们得到`call`调用enqueue方法时，实则是在调用`ExecutorCallbackCall`的enqueue方法。而enqueue中实则是在调用中传入的`OkHttpCall(delegate)`的enqueue方法。而这个OkHttpCall 是在`Retrofit.create`中创建的.

ExecutorCallbackCall当作是Wrapper，而真正去执行请求的源Source是OkHttpCall。之所以要有个Wrapper类，是希望在源Source操作时去做一些额外操作。这里的操作就是线程转换，将子线程切换到主线程上去。
enqueue()方法是异步的，也就是说，当你调用OkHttpCall的enqueue方法，回调的callback是在子线程中的，如果你希望在主线程接受回调，那需要通过Handler转换到主线程上去。ExecutorCallbackCall就是用来干这个事。当然以上是原生retrofit使用的切换线程方式。如果你用rxjava，那就不会用到这个ExecutorCallbackCall而是RxJava的Call了。最后流程图会有揭示

进入OkHttpCall的enqueue()
### OKHttpCall

```java
  @Override public void enqueue(final Callback<T> callback) {
    if (callback == null) throw new NullPointerException("callback == null");

    okhttp3.Call call;
    Throwable failure;
...
    call.enqueue(new okhttp3.Callback() {// 1.  调用OkHttp3类型的call的enqueue方法。
      @Override public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
          throws IOException {
        Response<T> response;
        try {
          response = parseResponse(rawResponse);
        } catch (Throwable e) {
          callFailure(e);
          return;
        }
        callSuccess(response);
      }

      @Override public void onFailure(okhttp3.Call call, IOException e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callFailure(Throwable e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callSuccess(Response<T> response) {
        try {
          callback.onResponse(OkHttpCall.this, response);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }
    });
  }
```
再进入parseResponse方法
```java
Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
    ResponseBody rawBody = rawResponse.body();
...
    int code = rawResponse.code();
    if (code < 200 || code >= 300) {
      try {
        // Buffer the entire body to avoid future I/O.
        ResponseBody bufferedBody = Utils.buffer(rawBody);
        return Response.error(bufferedBody, rawResponse);
      } finally {
        rawBody.close();
      }
    }

    if (code == 204 || code == 205) {
      rawBody.close();
      return Response.success(null, rawResponse);
    }

    ExceptionCatchingRequestBody catchingBody = new ExceptionCatchingRequestBody(rawBody);
    try {
      T body = serviceMethod.toResponse(catchingBody);
      return Response.success(body, rawResponse);
    } catch (RuntimeException e) {
      // If the underlying source threw an exception, propagate that rather than indicating it was
      // a runtime exception.
      catchingBody.throwIfCaught();
      throw e;
    }
  }
```
根据状态码进行对应操作，成功返回的情况下进入ServiceMethod.toResponse（）
<a name="GsonConverterFactory调用处" href = "#GsonConverterFactory声明处">GsonConverterFactory声明处</a>
```java
  R toResponse(ResponseBody body) throws IOException {
    return responseConverter.convert(body);
  }

```
通过此前提到在ServiceMethod的build()中获得了`GsonConverterFactory`
来完成数据转换。

## 总结
![design_pattern](/images/retrofit_stay.png)
