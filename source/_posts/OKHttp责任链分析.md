title: OKHttp责任链分析
date: 2018-3-23
tags: [Android]
categories: 源码解析
description: 我有一个加工资的梦想
---
参考自[okHttp责任链解析](https://www.jianshu.com/p/fd4255be60d7)
## 责任链概述
> 责任链模式：由多个处理请求的对象，其中每一个对象都引用了其下家的对象，从而形成一条处理请求的链。当客户端的请求到达时，并不
知道责任链上的哪一个对象处理了请求。这样的好处是：可以对客户端过来的请求做动态的处理。
>- 1.纯责任链模式：当请求到达一个类时要么处理完，要么扔给下一个对象。Sample：事件分发
>- 2.不纯的责任链模式：当请求到达时，该类处理一部分（可以配置一些参数，适配一下请求），然后扔给下一个。Sample：okhttp

## OKHttp责任链
OKHttp的责任链模式是一种不纯的责任链，即每一个拦截器都组装Reponse的一部分。

1.Interceptor接口中的intercept方法是实际处理请求的方法，每一个实现该接口的对象负责配置Request的一部分。并返回Response。
2.Chain接口负责执行intercept方法。并最终拿到intercept处理后的请求，绝大部分传递给intercept方法的Chain对象是RealInterceptorChain

```java
public interface Interceptor {
  Response intercept(Chain chain) throws IOException;

  interface Chain {
    Request request();

    Response proceed(Request request) throws IOException;

    Connection connection();
  }
}

```
1. proceed相当于责任链中的dispose方法，不过真正做实际工作的是它内部持有的interceptor对象的intercept方法。
2. Chain的proceed方法持有intercept对象，通过interceptors.get(index),并且将索引+1生成下一个chain传递给intercept方法。而当intercept生成一部分响应后又会调用proceed(实际调用intercept方法)。从而形成责任链。
从 getResponseWithInterceptorChain 开始分析
### getResponseWithInterceptorChain
```java
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);//1
    return chain.proceed(originalRequest);//2
  }

```
1. 在OkHttp的RealCall类中的getReponseWithInterceptorChain方法，配置了多个拦截器，组装成一个任务链对象(将配置好的拦截器数组，索引第一次是0，和请求)生成一个RealIntecceptorChain
2. 调用RealIntecceptorChain的proceed方法
### proceed
```java
//RealIntecceptorChain
  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      Connection connection) throws IOException {
   
    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);// 1
    Interceptor interceptor = interceptors.get(index);// 2
    Response response = interceptor.intercept(next);// 3
 ....
    return response;
  }

```
1.根据传进来的拦截器数组interceptors对象，和索引index构建一个新的拦截链---next对象。即每一个interceptor都会持有一个索引+1的任务链 RealInterceptorChain
2.从拦截器数组interceptors对象中根据索引获取拦截对象。并传入下一个任务链
3.调用拦截方法

开始拦截时，传入的索引是0，如果没有传入任何的interceptor类，那么将执行 RetryAndFollowUpInterceptor，调用的就是该类的intercept方法。查看代码。该类内部有两处也调用了chain.proceed方法。这个chain对象的持有的索引是(当前是0)index+1。调用它的proceed方法会执行集合中index=1的interceptor对象。

### RetryAndFollowUpInterceptor
```java
// RetryAndFollowUpInterceptor
  @Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();

    streamAllocation = new StreamAllocation(
        client.connectionPool(), createAddress(request.url()), callStackTrace);// 1

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {

      Response response = null;
      boolean releaseConnection = true;
      try {
        response = ((RealInterceptorChain) chain).proceed(request, streamAllocation, null, null); // 2
        releaseConnection = false;
      } 
	  ...
        return response;// 3
      }
```
1. 配置request的失败重试以及重定向功能
2. 调用剩下的拦截链
3. 返回response
进入 BridgeInterceptor
### BridgeInterceptor
```java
//BridgeInterceptor
  @Override public Response intercept(Chain chain) throws IOException {
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();

    RequestBody body = userRequest.body();
    if (body != null) {
      MediaType contentType = body.contentType();
      if (contentType != null) {
        requestBuilder.header("Content-Type", contentType.toString());
      }

      long contentLength = body.contentLength();
      if (contentLength != -1) {
        requestBuilder.header("Content-Length", Long.toString(contentLength));
        requestBuilder.removeHeader("Transfer-Encoding");
      } else {
        requestBuilder.header("Transfer-Encoding", "chunked");
        requestBuilder.removeHeader("Content-Length");
      }
    }

    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", hostHeader(userRequest.url(), false));
    }

    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }

    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }

    List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
    if (!cookies.isEmpty()) {
      requestBuilder.header("Cookie", cookieHeader(cookies));
    }

    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }
// 1
    Response networkResponse = chain.proceed(requestBuilder.build());//2

    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());

    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);

    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      responseBuilder.body(new RealResponseBody(strippedHeaders, Okio.buffer(responseBody)));
    }

    return responseBuilder.build();// 3
  }
```
1. 负责把用户构造的请求转换为发送到服务器的请求
2. 调用剩余的拦截链处理请求
3. 把服务器返回的Response转换为用户友好的Response返回

依次类推，直到集合中的所有拦截器的interceptor方法全部调用完毕，流程是procced---interceptor---procced---interceptor---procced----interceptor---interceptor方法。完成整个任务链。
## 加工资
比如加工资审批场景

```java

public interface Interceptor {
    public void intercept(Chain chain);

    interface Chain {
        public void proceed(int salary, String name);
    }
}

```
```java
//hr拦截审批
class StaffInterceptor implements Interceptor {
    @Override
    public void intercept(Chain chain) {
        String user = ((RealInterceptorChain)chain).getUser();
        int salary = ((RealInterceptorChain)chain).getSalary() + 300;

        System.out.println(user + " Hr审核通过 : " + salary);

        chain.proceed(salary, user);
    }
}
//经理拦截审批
class ManagerInterceptor implements Interceptor {
    @Override
    public void intercept(Chain chain) {
        String user = ((RealInterceptorChain)chain).getUser();
        int salary = ((RealInterceptorChain)chain).getSalary() + 300;

        System.out.println(user  + " Manager审核通过 : " + salary);

        chain.proceed(salary, user);
    }
}
```

```java
//拦截链
class RealInterceptorChain implements Interceptor.Chain {

    private List<Interceptor> mInterceptors;
    private int mIndex;
    private String mUser;
    private int mSalary;

    public RealInterceptorChain(List<Interceptor> interceptors, int index) {
        mInterceptors = interceptors;
        mIndex = index;
    }


    @Override
    public void proceed(int salary, String user) {
        this.mSalary = salary;
        this.mUser = user;
        if (mIndex >= mInterceptors.size()) {
            return;
        }
        RealInterceptorChain nextChain = this;


        Interceptor interceptor = nextChain.mInterceptors.get(mIndex);

        nextChain.setIndex(mIndex + 1);
        interceptor.intercept(nextChain);

    }


    public String getUser() {
        return mUser;
    }

    public int getSalary() {
        return mSalary;
    }

    public void setIndex(int index) {
        mIndex = index;
    }

    public static void main(String[] args) {
        List<Interceptor> interceptors = new ArrayList<>();
        interceptors.add(new StaffInterceptor());
        interceptors.add(new ManagerInterceptor());

        RealInterceptorChain chain = new RealInterceptorChain(interceptors, 0);

        chain.proceed(6000, "night");
    }
}
```
