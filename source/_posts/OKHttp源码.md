title: OKHttp源码
date: 2018-3-23
tags: [Android]
categories: 源码解析
description: 啊，好长的一坨代码呐
---
参考自：[okhttp解析](https://blog.piasy.com/2016/07/11/Understand-OkHttp/index.html)

## OkHttp源码分析
![enter description here](/images/okhttp_full_process.png)
首先放一张完整流程图与样例
```java
//demo
//sync
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();

  try (Response response = client.newCall(request).execute()) {
    return response.body().string();
  }
}
//async
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
    }
    @Override
    public void onResponse(Call call, Response response) throws IOException {
        System.out.println(response.body().string());
    }
});
```
### 创建OkHttpClient对象
```java
OkHttpClient client = new OkHttpClient();
```
进入构造方法
```java
public OkHttpClient() {
  this(new Builder());
}
public Builder() {
  dispatcher = new Dispatcher();
  protocols = DEFAULT_PROTOCOLS;
  connectionSpecs = DEFAULT_CONNECTION_SPECS;
  proxySelector = ProxySelector.getDefault();
  cookieJar = CookieJar.NO_COOKIES;
  socketFactory = SocketFactory.getDefault();
  hostnameVerifier = OkHostnameVerifier.INSTANCE;
  certificatePinner = CertificatePinner.DEFAULT;
  proxyAuthenticator = Authenticator.NONE;
  authenticator = Authenticator.NONE;
  connectionPool = new ConnectionPool();
  dns = Dns.SYSTEM;
  followSslRedirects = true;
  followRedirects = true;
  retryOnConnectionFailure = true;
  connectTimeout = 10_000;
  readTimeout = 10_000;
  writeTimeout = 10_000;
}
```
这里采用了建造者模式，动态添加对象属性，提高代码可读性。 

### 发起Http请求
```java
String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();
  Response response = client.newCall(request).execute();
  return response.body().string();
}
```
OkHttpClient实现了Call.Factory，负责根据请求创建新的Call。进入newCall
```java
/**
  * Prepares the {@code request} to be executed at some point in the future.
  */
@Override public Call newCall(Request request) {
  return new RealCall(this, request);
}
```
进入RealCall

### 同步网络请求
RealCall`execute()`
```java
@Override public Response execute() throws IOException {
  synchronized (this) {
    if (executed) throw new IllegalStateException("Already Executed");  // (1)
    executed = true;
  }
  try {
    client.dispatcher().executed(this);                                 // (2)
    Response result = getResponseWithInterceptorChain();                // (3)
    if (result == null) throw new IOException("Canceled");
    return result;
  } finally {
    client.dispatcher().finished(this);                                 // (4)
  }
}
```
1. 检查这个 call 是否已经被执行了，每个 call 只能被执行一次，如果想要一个完全一样的 call，可以利用call#clone方法进行克隆。
2. 利用client.dispatcher().executed(this)来进行实际执行dispatcher是刚才看到的OkHttpClient.Builder的成员之一，文档说自己是异步 HTTP 请求的执行策略，现在看来，同步请求它也有掺和。
3. 调用getResponseWithInterceptorChain()函数获取 HTTP 返回结果，从函数名可以看出，这一步还会进行一系列“拦截”操作。
4. 最后还要通知dispatcher自己已经执行完毕。

dispatcher 这里我们不过度关注，在同步执行的流程中，涉及到 dispatcher 的内容只不过是告知它我们的执行状态，比如开始执行了（调用executed），比如执行完毕了（调用finished），在异步执行流程中它会有更多的参与。

真正发出网络请求，解析返回结果的，还是getResponseWithInterceptorChain：
```java
private Response getResponseWithInterceptorChain() throws IOException {
  // Build a full stack of interceptors.
  List<Interceptor> interceptors = new ArrayList<>();
  interceptors.addAll(client.interceptors());
  interceptors.add(retryAndFollowUpInterceptor);
  interceptors.add(new BridgeInterceptor(client.cookieJar()));
  interceptors.add(new CacheInterceptor(client.internalCache()));
  interceptors.add(new ConnectInterceptor(client));
  if (!retryAndFollowUpInterceptor.isForWebSocket()) {
    interceptors.addAll(client.networkInterceptors());
  }
  interceptors.add(new CallServerInterceptor(
      retryAndFollowUpInterceptor.isForWebSocket()));
  Interceptor.Chain chain = new RealInterceptorChain(
      interceptors, null, null, null, 0, originalRequest);
  return chain.proceed(originalRequest);
}
```

> the whole thing is just a stack of built-in interceptors.

`Interceptor`是 OkHttp 最核心的一个东西，不要误以为它只负责拦截请求进行一些额外的处理（例如 cookie），__实际上它把实际的网络请求、缓存、透明压缩等功能都统一了起来__，`每一个功能都只是一个Interceptor`，它们再连接成一个`Interceptor.Chain`，如链条一般,环环相扣，最终圆满完成一次网络请求。关于责任链模式在下一篇博文有详细阐述。

从`getResponseWithInterceptorChain`函数我们可以看到`Interceptor.Chain`的分布依次是：
![okhttp_interceptors](/images/okhttp_interceptors.png)
1. 配置`OkHttpClient`时设置的`interceptors`；
2. 负责失败重试以及重定向的`RetryAndFollowUpInterceptor`；
3. 负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应的`BridgeInterceptor`；
4. 负责读取缓存直接返回、更新缓存的`CacheInterceptor`；
5. 负责和服务器建立连接的`ConnectInterceptor`；
6. 配置`OkHttpClient`时设置的`networkInterceptors`, interceptors that observe a single network request and response.
7. 负责向服务器发送请求数据、从服务器读取响应数据`CallServerInterceptor`

位置决定了功能，最后一个 Interceptor 一定是负责和服务器实际通讯的，重定向、缓存等一定是在实际通讯之前的。

对于把`Request`变成`Response`这件事来说，每个`Interceptor`都可能完成这件事，所以我们循着链条让每个`Interceptor`自行决定能否完成任务以及怎么完成任务（自力更生或者交给下一个`Interceptor`）。这样一来，完成网络请求这件事就彻底从`RealCall`类中剥离了出来，简化了各自的责任和逻辑。

进入`ConnectInterceptor和CallServerInterceptor`，看看 OkHttp 是怎么进行和服务器的实际通信的。
### 建立连接：ConnectInterceptor
```java
@Override public Response intercept(Chain chain) throws IOException {
  RealInterceptorChain realChain = (RealInterceptorChain) chain;
  Request request = realChain.request();
  StreamAllocation streamAllocation = realChain.streamAllocation();
  // We need the network to satisfy this request. Possibly for validating a conditional GET.
  boolean doExtensiveHealthChecks = !request.method().equals("GET");
  HttpCodec httpCodec = streamAllocation.newStream(client, doExtensiveHealthChecks);
  RealConnection connection = streamAllocation.connection();
  return realChain.proceed(request, streamAllocation, httpCodec, connection);
}
```
建立连接就是创建了一个`HttpCodec`对象，它将在后面的步骤中被使用，那它又是何方神圣呢？它是对 HTTP 协议操作的抽象，有两个实现：`Http1Codec和Http2Codec`，顾名思义，它们分别对应 HTTP/1.1 和 HTTP/2 版本的实现。

在Http1Codec中，它利用Okio对Socket的读写操作进行封装，Okio 以后有机会再进行分析，现在让我们对它们保持一个简单地认识：它对java.io和java.nio进行了封装，让我们更便捷高效的进行 IO 操作。

而创建`HttpCodec`对象的过程涉及到`StreamAllocation、RealConnection`，代码较长，这里就不展开，这个过程概括来说，就是找到一个`可用的RealConnection`，再利用RealConnection的输入输出（BufferedSource和BufferedSink）创建HttpCodec对象，供后续步骤使用。

### 发送和接收数据：CallServerInterceptor
```java
@Override public Response intercept(Chain chain) throws IOException {
  HttpCodec httpCodec = ((RealInterceptorChain) chain).httpStream();
  StreamAllocation streamAllocation = ((RealInterceptorChain) chain).streamAllocation();
  Request request = chain.request();
  long sentRequestMillis = System.currentTimeMillis();
  httpCodec.writeRequestHeaders(request);
  if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
    Sink requestBodyOut = httpCodec.createRequestBody(request, request.body().contentLength());
    BufferedSink bufferedRequestBody = Okio.buffer(requestBodyOut);
    request.body().writeTo(bufferedRequestBody);
    bufferedRequestBody.close();
  }
  httpCodec.finishRequest();
  Response response = httpCodec.readResponseHeaders()
      .request(request)
      .handshake(streamAllocation.connection().handshake())
      .sentRequestAtMillis(sentRequestMillis)
      .receivedResponseAtMillis(System.currentTimeMillis())
      .build();
  if (!forWebSocket || response.code() != 101) {
    response = response.newBuilder()
        .body(httpCodec.openResponseBody(response))
        .build();
  }
  if ("close".equalsIgnoreCase(response.request().header("Connection"))
      || "close".equalsIgnoreCase(response.header("Connection"))) {
    streamAllocation.noNewStreams();
  }
  // 省略部分检查代码
  return response;
}
```
1. 向服务器发送 request header；
2. 如果有 request body，就向服务器发送；
3. 读取 response header，先构造一个Response对象；
4. 如果有 response body，就在 3 的基础上加上 body 构造一个新的Response对象；

核心工作都由HttpCodec对象完成，而HttpCodec实际上利用的是 Okio，而 Okio 实际上还是用的Socket，所以没什么神秘的，只不过一层套一层，层数有点多。

其实Interceptor的设计也是一种分层的思想，每个Interceptor就是一层。为什么要套这么多层呢？分层的思想在 TCP/IP 协议中就体现得淋漓尽致，分层简化了每一层的逻辑，每层只需要关注自己的责任（单一原则思想也在此体现），而各层之间通过约定的接口/协议进行合作（面向接口编程思想），共同完成复杂的任务。

### 异步网络请求
```java
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
    }
    @Override
    public void onResponse(Call call, Response response) throws IOException {
        System.out.println(response.body().string());
    }
});
// RealCall的enqueue
@Override public void enqueue(Callback responseCallback) {
  synchronized (this) {
    if (executed) throw new IllegalStateException("Already Executed");
    executed = true;
  }
  client.dispatcher().enqueue(new AsyncCall(responseCallback));
}
// Dispatcher的enqueue
synchronized void enqueue(AsyncCall call) {
  if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
    runningAsyncCalls.add(call);
    executorService().execute(call);
  } else {
    readyAsyncCalls.add(call);
  }
}
```
在dispatcher的enqueue（）中，如果当前队列大小 小于64，且队列内当前请求的同一host不超过5个，则直接加入队列开始执行，否则只加入队列。
这里的AsyncCall是RealCall的一个内部类，它实现了Runnable，所以可以被提交到ExecutorService（线程池）上执行，而它在执行时会调用getResponseWithInterceptorChain()函数，并把结果通过responseCallback传递给上层使用者。

这样看来，同步请求和异步请求的原理是一样的，都是在getResponseWithInterceptorChain()函数中通过Interceptor链条来实现的网络请求逻辑，而异步则是通过ExecutorService实现。

### 返回数据的获取
上述同步（Call#execute()执行之后）或者异步（Callback#onResponse()回调中）请求完成之后，我们就可以从Response对象中获取到响应数据了，包括 HTTP status code，status message，response header，response body 等。这里 body 部分最为特殊，因为服务器返回的数据可能非常大，所以必须通过数据流的方式来进行访问（当然也提供了诸如string()和bytes()这样的方法将流内的数据一次性读取完毕），而响应中其他部分则可以随意获取。

响应 body 被封装到ResponseBody类中，该类主要有两点需要注意：

1. 每个 body 只能被消费一次，多次消费会抛出异常；
2. body 必须被关闭，否则会发生资源泄漏；
在发送和接收数据：CallServerInterceptor小节中，我们就看过了 body 相关的代码。
```java
if (!forWebSocket || response.code() != 101) {
  response = response.newBuilder()
      .body(httpCodec.openResponseBody(response))
      .build();
}
```
由`HttpCodec#openResponseBody`提供具体 HTTP 协议版本的响应 body，而`HttpCodec`则是利用 Okio 实现具体的数据 IO 操作。

### Http缓存（参见下一篇博文）
我们已经看到了`Interceptor`的布局，在建立连接、和服务器通讯之前，就是`CacheInterceptor`，在建立连接之前，我们检查响应是否已经被缓存、缓存是否可用，如果是则直接返回缓存的数据，否则就进行后面的流程，并在返回之前，把网络的数据写入缓存。

这块代码比较多，但也很直观，主要涉及 HTTP 协议缓存细节的实现，而具体的缓存逻辑 OkHttp 内置封装了一个`Cache`类，它利用`DiskLruCache`，用磁盘上的有限大小空间进行缓存，按照 LRU 算法进行缓存淘汰。
### 连接复用
若有大量连接，每次都要建立连接，释放连接，这会导致性能低下，在Http中有keepalive connections机制，它可以在传输数据后保持连接，当客户端再次获取数据时，直接使用刚刚空闲下来的连接而无须再次握手。
![okhttp_reuse](/images/connect_reuse.png)

1. 建立连接并保存至连接池connections（Deque < RealConnection >）中
2. 写请求数据
3. 读相应数据
4. 计算延时timeout5分钟，延时内若有数据请求访问，遍历连接池如果连接池请求具有相同Address（相同的Host与端口）可以共用相同的连接RealConnection。
5. 回到2.，否则进入5
6. OkHttp根据StreamAllocation引用计数是否为0来实现自动回收连接


## 总结
![enter description here](/images/okhttp_full_process.png)
OkHttp执行流程如下：
1. OkHttpClient实现Call.Factory，负责为Request创建Call；
2. RealCall为具体的Call实现，其enqueue()异步接口通过Dispatcher利用ExecutorService实现，而最终进行网络请求时和同步execute()接口一致，都是通过getResponseWithInterceptorChain()函数实现；
3. getResponseWithInterceptorChain()中利用Interceptor链条，分层实现缓存、透明压缩、网络 IO 等功能；