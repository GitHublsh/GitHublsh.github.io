---
title: OkHttp3源码学习（3）-拦截器链详解
date: 2017-07-27 17:02:28
tags: [OkHttp3]
---
#### 一、发起请求

```
OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url("")
                .build();
        Call call = client.newCall(request);
        try {
            call.enqueue(new okhttp3.Callback() {
                @Override
                public void onFailure(Call call, IOException e) {
                    Log.d("OkHttp", "Call Failed:" + e.getMessage());
                }

                @Override
                public void onResponse(Call call, Response response) throws IOException {
                    Log.d("OkHttp", "Call succeeded:" + response.message());
                }
            });
        } catch (Exception e) {
            Log.e("OkHttp",e.getMessage());
        }
```
	        
* 发起请求时：client.newCall(request)。

```
@Override public Call newCall(Request request) {
	    return new RealCall(this, request, false /* for web socket */);
	  	}
```
  
  实际上就是创建一个RealCall的实例。
  
*  然后call.enqueue,源码实现就是将RealCall加到任务队列中，等合适的机会去执行。

```
@Override public void enqueue(Callback responseCallback) {
	    synchronized (this) {
	      if (executed) throw new IllegalStateException("Already Executed");
	      executed = true;
	    }
	    captureCallStackTrace();
	    client.dispatcher().enqueue(new AsyncCall(responseCallback));
	  	}
```

#### 二、AsyncCall

```
@Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
```

AsyncCall会执行execute方法。execute方法逻辑很简单：
  

* 通过调用getResponseWithInterceptorChain获取服务器返回结果，失败或者成功

```
Response response = getResponseWithInterceptorChain();
```

* 通知任务分发器该任务结束

```
client.dispatcher().finished(this);
```
     
#### 三、构建拦截器链


```
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
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }
```	

从源码来看，基本逻辑就是：

1.创建一系列拦截器，加到拦截器数组中。
2.创建拦截器链RealInterceptorChain
3.执行拦截器链中的proceed方法

#### 四、RealInterceptorChain


```
public final class RealInterceptorChain implements Interceptor.Chain {
	  private final List<Interceptor> interceptors;
	  private final StreamAllocation streamAllocation;
	  private final HttpCodec httpCodec;
	  private final RealConnection connection;
	  private final int index;
	  private final Request request;
	  private int calls;
	
	  public RealInterceptorChain(List<Interceptor> interceptors, StreamAllocation streamAllocation,
	      HttpCodec httpCodec, RealConnection connection, int index, Request request) {
	    this.interceptors = interceptors;
	    this.connection = connection;
	    this.streamAllocation = streamAllocation;
	    this.httpCodec = httpCodec;
	    this.index = index;
	    this.request = request;
	  }
	
	  @Override public Connection connection() {
	    return connection;
	  }
	
	  public StreamAllocation streamAllocation() {
	    return streamAllocation;
	  }
	
	  public HttpCodec httpStream() {
	    return httpCodec;
	  }
	
	  @Override public Request request() {
	    return request;
	  }
	
	  @Override public Response proceed(Request request) throws IOException {
	    return proceed(request, streamAllocation, httpCodec, connection);
	  }
	
	  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
	      RealConnection connection) throws IOException {
	    if (index >= interceptors.size()) throw new AssertionError();
	
	    calls++;
	
	    // If we already have a stream, confirm that the incoming request will use it.
	    if (this.httpCodec != null && !this.connection.supportsUrl(request.url())) {
	      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
	          + " must retain the same host and port");
	    }
	
	    // If we already have a stream, confirm that this is the only call to chain.proceed().
	    if (this.httpCodec != null && calls > 1) {
	      throw new IllegalStateException("network interceptor " + interceptors.get(index - 1)
	          + " must call proceed() exactly once");
	    }
	
	    

	    Interceptor interceptor = interceptors.get(index);
	    Response response = interceptor.intercept(next);
	
	    // Confirm that the next interceptor made its required call to chain.proceed().
	    if (httpCodec != null && index + 1 < interceptors.size() && next.calls != 1) {
	      throw new IllegalStateException("network interceptor " + interceptor
	          + " must call proceed() exactly once");
	    }
	
	    // Confirm that the intercepted response isn't null.
	    if (response == null) {
	      throw new NullPointerException("interceptor " + interceptor + " returned null");
	    }
	
	    return response;
	  }
	}
```

可以看到procees方法的逻辑：

创建下一个拦截链（代码中的next），传入index+1，使创建的下一个拦截器链只能从下一个拦截器访问。


```
RealInterceptorChain next = new RealInterceptorChain(
			        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
```
		        
* 获取索引为index的interceptor,执行索引为index的intercept方法。


```
Interceptor interceptor = interceptors.get(index);
Response response = interceptor.intercept(next);
```


#### 五、拦截器链

##### 1.RetryAndFollowUpInterceptor

```
@Override 
public Response intercept(Chain chain) throws IOException {
	    Request request = chain.request();
	
    streamAllocation = new StreamAllocation(
        client.connectionPool(), createAddress(request.url()), callStackTrace);
	
    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      if (canceled) {
        streamAllocation.release();
        throw new IOException("Canceled");
      }
	
      Response response = null;
      boolean releaseConnection = true;
      try {
        response = ((RealInterceptorChain) chain).proceed(request, streamAllocation, null, null);
        releaseConnection = false;
      } catch (RouteException e) {
        // The attempt to connect via a route failed. The request will not have been sent.
        if (!recover(e.getLastConnectException(), false, request)) {
          throw e.getLastConnectException();
        }
        releaseConnection = false;
        continue;
      } catch (IOException e) {
        // An attempt to communicate with a server failed. The request may have been sent.
        boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
        if (!recover(e, requestSendStarted, request)) throw e;
        releaseConnection = false;
        continue;
      } finally {
        // We're throwing an unchecked exception. Release any resources.
        if (releaseConnection) {
          streamAllocation.streamFailed(null);
          streamAllocation.release();
        }
      }
	
      // Attach the prior response if it exists. Such responses never have a body.
      if (priorResponse != null) {
        response = response.newBuilder()
            .priorResponse(priorResponse.newBuilder()
                    .body(null)
                    .build())
            .build();
      }
	
      Request followUp = followUpRequest(response);
	
      if (followUp == null) {
        if (!forWebSocket) {
          streamAllocation.release();
        }
        return response;
      }
	
      closeQuietly(response.body());
	
      if (++followUpCount > MAX_FOLLOW_UPS) {
        streamAllocation.release();
        throw new ProtocolException("Too many follow-up requests: " + followUpCount);
      }
	
      if (followUp.body() instanceof UnrepeatableRequestBody) {
        streamAllocation.release();
        throw new HttpRetryException("Cannot retry streamed HTTP body", response.code());
      }
	//与当前的响应进行对比，检查是否同一个连接。通常，当发生请求重定向时，url地址将会有所不同，也就是说，
	请求的资源在这时已经被分配了新的url.当不是同一个url请求时，将原先的streamAllocation执行release销
	毁掉,再新建一个StreamAllocation连接,进行重试。
      if (!sameConnection(response, followUp.url(s))) {
        streamAllocation.release();
        streamAllocation = new StreamAllocation(
            client.connectionPool(), createAddress(followUp.url()), callStackTrace);
      } else if (streamAllocation.codec() != null) {
        throw new IllegalStateException("Closing the body of " + response
            + " didn't close its backing stream. Bad interceptor?");
      }
	
      request = followUp;
      priorResponse = response;
    }
	}
```	
	
	
1.发起请求前拦截器对request处理
2.然后调用下一个拦截器，获取Response

调用的关键：

```
try {
    response = ((RealInterceptorChain) chain).proceed(request, streamAllocation, null, null);
    releaseConnection = false;
}
```

那么这个时候就会去调用下一个拦截器。对response进行处理，返回给上一个拦截器.





下面就来对几种拦截器一一介绍。

##### 2.BidgeInterceptor

官方注释解释：从应用程序代码到网络代码的桥梁，首先从用户的请求构建一个网络请求，然后执行访问网络，最后返回Response.

整个过程就是：首先将客户端构建的Request对象信息构建成真正的网络请求;然后发起网络请求，最后就是将服务器返回的消息封装成一个Response对象。

下面就看一下核心方法intercept()

```
@Override 
public Response intercept(Chain chain) throws IOException {
	//拿到用户的请求
    Request userRequest = chain.request();
    Request.Builder requestBuilder = userRequest.newBuilder();
	//拿到用户请求body
    RequestBody body = userRequest.body();
    //对请求头的补充
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
	//默认是保持连接的（Keep-Alive）
    if (userRequest.header("Connection") == null) {
      requestBuilder.header("Connection", "Keep-Alive");
    }
	//默认GZIP压缩
	//Accept-Encoding就是告诉服务器客户端能接收的数据编码类型
    // If we add an "Accept-Encoding: gzip" header field we're responsible for also decompressing
    // the transfer stream.
    boolean transparentGzip = false;
    if (userRequest.header("Accept-Encoding") == null && userRequest.header("Range") == null) {
      transparentGzip = true;
      requestBuilder.header("Accept-Encoding", "gzip");
    }
	//添加cookie头
    List<Cookie> cookies = cookieJar.loadForRequest(userRequest.url());
    if (!cookies.isEmpty()) {
      requestBuilder.header("Cookie", cookieHeader(cookies));
    }

    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }
	//继续执行下一个拦截器的方法
    Response networkResponse = chain.proceed(requestBuilder.build());
	//接收服务器返回的cookie
    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());

    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);

    if (transparentGzip
        && "gzip".equalsIgnoreCase(networkResponse.header("Content-Encoding"))
        && HttpHeaders.hasBody(networkResponse)) {
      //当服务器返回的数据是GZIP压缩的，那么客户端就进行GZIP解压操作
      GzipSource responseBody = new GzipSource(networkResponse.body().source());
      Headers strippedHeaders = networkResponse.headers().newBuilder()
          .removeAll("Content-Encoding")
          .removeAll("Content-Length")
          .build();
      responseBuilder.headers(strippedHeaders);
      responseBuilder.body(new RealResponseBody(strippedHeaders, Okio.buffer(responseBody)));
    }
	//构建一个Response
    return responseBuilder.build();
  	}
```

BridgeInterceptor主要流程逻辑：

1.拿到用户的请求,将用户的构建的Request请求转化为真正的网络请求
2.将这个符合网络请求的Request进行网络请求
3.将网络请求返回的Response转化为用户可用的Response
	
	
代码中构建网络Request添加的请求头信息：

* 简单了解一下先，头信息

协议头字段名 |说明| 示例 | 状态
---- | --- | --- | ---
Content-Type | 请求体的 多媒体类型 （用于POST和PUT请求中）|Content-Type: application/x-www-form-urlencoded|常设
Content-Length |  以 八位字节数组 （8位的字节）表示的请求体的长度|Content-Length: 348|常设
Transfer-Encoding|用来将实体安全地传输给用户的编码形式。当前定义的方法包括：分块（chunked）、compress、deflate、gzip和identity|Transfer-Encoding: chunked|常设
Host|服务器的域名(用于虚拟主机 )，以及服务器所监听的传输控制协议端口号。如果所请求的端口是对应的服务的标准端口，则端口号可被省略。自超文件传输协议版本1.1（HTTP/1.1）开始便是必需字段。| Host: en.wikipedia.org:80 Host: en.wikipedia.org|常设
Connection|该浏览器想要优先使用的连接类型|Connection: keep-alive  	，Connection: Upgrade|常设
Accept-Encoding|能够接受的编码方式列表|Accept-Encoding: gzip, deflate|常设
Cookie|之前由服务器通过 Set- Cookie （下文详述）发送的一个 超文本传输协议Cookie。指某些网站为了辨别用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密）。定义于RFC2109|Cookie: $Version=1; Skin=new;|常设: 标准

###### 小结：

构建完头信息后，进行网络请求

```
Response networkResponse = chain.proceed(requestBuilder.build());
```
		
获取到返回的Response,转化为客户端可用的Response

```
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
```

##### 3.CacheIntetceptor

CacheIntetceptor的职责就是负责Cache的管理

看一下核心方法：

```
@Override public Response intercept(Chain chain) throws IOException {
	 //1.读取候选的缓存
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();
	//2.首先创建缓存策略，networkRequest为网络请求，cacheResponse为缓存
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      cache.trackResponse(strategy);
    }

    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    //3.如果禁止网络访问并且本地cache缓存也不完整，那么请求失败
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    //4.不需要访问网络的情况下，取本地缓存作为结果返回。
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    Response networkResponse = null;
    try {
    //5.当以上情况都没有结果返回，就读取网络结果（继续执行下一个拦截器）
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    //6.接收到网络结果返回，如果我们也有缓存，那么就会进行条件对比组合
    if (cacheResponse != null) {
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))//7.将缓存返回与网络返回的头信息进行组合
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();
		//8.组合头后，但在剥离Content-Encoding头（由initContentStream（）执行）之前更新缓存。
        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        cache.update(cacheResponse, response);
        return response;
      } else {
        closeQuietly(cacheResponse.body());
      }
    }
	//9.读取网络请求
    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();
	//10.对数据进行缓存
    if (cache != null) {
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // Offer this request to the cache.
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }
	//11.返回网络请求的结果
    return response;
  	}
```	
	
####### 整个过程大致：

CacheInterceptor主要就是负责Cache的管理

1.当网络被禁止访问，缓存不完整，那么返回失败（504）
2.缓存可用，返回缓存结果
3.当网络访问，返回（304），更新本地缓存
4.当Cache失效，删除缓存

  
##### 4.ConnectInterceptor

代码不多，但包含的内容很多。

```
@Override
public Response intercept(Chain chain) throws IOException {
	    RealInterceptorChain realChain = (RealInterceptorChain) chain;
	    Request request = realChain.request();
	    //拿到StreamAllocation对象。
	    StreamAllocation streamAllocation = realChain.streamAllocation();
	
	    // We need the network to satisfy this request. Possibly for validating a conditional GET.
	    boolean doExtensiveHealthChecks = !request.method().equals("GET");
	    HttpCodec httpCodec = streamAllocation.newStream(client, doExtensiveHealthChecks);
	    RealConnection connection = streamAllocation.connection();
	
	    return realChain.proceed(request, streamAllocation, httpCodec, connection);
	  	}
```	
	
从源码来看，StreamAllocation在RetryAndFollowUpInterceptor中进行的初始化

```
streamAllocation = new StreamAllocation(
        client.connectionPool(), createAddress(request.url()), callStackTrace);
```
        
三个参数分别是：一个连接池，一个地址类，一个调用堆栈跟踪相关。主要是把这个三个参数保存为内部变量，供后面使用

看一下StreamAllocation的构造方法

```
public StreamAllocation(ConnectionPool connectionPool, Address address, Object callStackTrace) {
    this.connectionPool = connectionPool;
    this.address = address;
    this.routeSelector = new RouteSelector(address, routeDatabase());
    this.callStackTrace = callStackTrace;
  	}
```

在把这个三个参数保存为内部变量的同时也创建了一个线路选择器

streamAllocation.newStream 通过这个方法得到一个 HttpStream 这个接口有两个实现类分别是 Http1xStream 和 Http2xStream 现在只分析 Http1xStream ，这个 Http1xStream 流是通过 SOCKET 与服务端建立连接之后，通向服务端的输入和输出流的封装。

接下来继续看StreamAllocation中的newSream()方法

```
public HttpCodec newStream(OkHttpClient client, boolean doExtensiveHealthChecks) {
	//读取从OkHttpClient配置的超时时间
    int connectTimeout = client.connectTimeoutMillis();
    //获取读写超时时间
    int readTimeout = client.readTimeoutMillis();
    int writeTimeout = client.writeTimeoutMillis();
    //连接重试
    boolean connectionRetryEnabled = client.retryOnConnectionFailure();

    try {
    //找到一个健康的连接（在连接池中寻找或者新创建一个连接）
      RealConnection resultConnection = findHealthyConnection(connectTimeout, readTimeout,
          writeTimeout, connectionRetryEnabled, doExtensiveHealthChecks);
      //HttpCodec用来编码HTTP请求并解码HTTP响应。在这里初始化
      HttpCodec resultCodec = resultConnection.newCodec(client, this);

      synchronized (connectionPool) {
        codec = resultCodec;
        return resultCodec;
      }
    } catch (IOException e) {
      throw new RouteException(e);
    }
  	}
```
  	
 下面就再看一下它是如何找到一个健康的连接的

```
private RealConnection findHealthyConnection(int connectTimeout, int readTimeout,
	      int writeTimeout, boolean connectionRetryEnabled, boolean doExtensiveHealthChecks)
	      throws IOException {
	    while (true) {
	    //找到健康的连接
	      RealConnection candidate = findConnection(connectTimeout, readTimeout, writeTimeout,
	          connectionRetryEnabled);
	
	      // If this is a brand new connection, we can skip the extensive health checks.
	      synchronized (connectionPool) {
	        if (candidate.successCount == 0) {
	          return candidate;
	        }
	      }
	
	      // Do a (potentially slow) check to confirm that the pooled connection is still good. If it
	      // isn't, take it out of the pool and start again.
	      if (!candidate.isHealthy(doExtensiveHealthChecks)) {
	        noNewStreams();
	        continue;
	      }
	
	      return candidate;
	    }
	  }
```	
	  
从源码来看，这个方法就是找到一个连接并返回它，如果它是健康的。 如果这是不健康的，那么这个过程将被重复，直到找到一个健康的连接。

那么继续跟进，看一下是怎么找到健康的连接，进入findConnection(connectTimeout,readTimeout, writeTimeout,connectionRetryEnabled)方法
	          
```
private RealConnection findConnection(int connectTimeout, int readTimeout, int writeTimeout,
	      boolean connectionRetryEnabled) throws IOException {
	    Route selectedRoute;
	    //同步线程池
	    synchronized (connectionPool) {
	      if (released) throw new IllegalStateException("released");
	      if (codec != null) throw new IllegalStateException("codec != null");
	      if (canceled) throw new IOException("Canceled");
	
			//尝试使用现有连接，判断是否可用
	      // Attempt to use an already-allocated connection.
	      RealConnection allocatedConnection = this.connection;
	      if (allocatedConnection != null && !allocatedConnection.noNewStreams) {
	        return allocatedConnection;
	      }
			//尝试在连接池中获取一个连接，
	      // Attempt to get a connection from the pool.
	      Internal.instance.get(connectionPool, address, this, null);
	      if (connection != null) {
	        return connection;
	      }
	
	      selectedRoute = route;
	    }
	
	    // If we need a route, make one. This is a blocking operation.
	    if (selectedRoute == null) {
	      selectedRoute = routeSelector.next();
	    }
	
	    RealConnection result;
	    synchronized (connectionPool) {
	      if (canceled) throw new IOException("Canceled");
	
	      // Now that we have an IP address, make another attempt at getting a connection from the pool.
	      // This could match due to connection coalescing.
	      Internal.instance.get(connectionPool, address, this, selectedRoute);
	      if (connection != null) {
	        route = selectedRoute;
	        return connection;
	      }
	
	      // Create a connection and assign it to this allocation immediately. This makes it possible
	      // for an asynchronous cancel() to interrupt the handshake we're about to do.
	      route = selectedRoute;
	      refusedStreamCount = 0;
	      result = new RealConnection(connectionPool, selectedRoute);
	      acquire(result);
	    }
	
	    // Do TCP + TLS handshakes. This is a blocking operation.
	    result.connect(connectTimeout, readTimeout, writeTimeout, connectionRetryEnabled);
	    routeDatabase().connected(result.route());
	
	    Socket socket = null;
	    synchronized (connectionPool) {
	      // Pool the connection.
	      Internal.instance.put(connectionPool, result);
	
	      // If another multiplexed connection to the same address was created concurrently, then
	      // release this connection and acquire that one.
	      if (result.isMultiplexed()) {
	        socket = Internal.instance.deduplicate(connectionPool, address, this);
	        result = connection;
	      }
	    }
	    closeQuietly(socket);
	
	    return result;
	  }
```       

这个方法的大致逻辑就是：返回连接以托管新流。 如果现有的连接存在，则优先选择池，最后建立一个新的连接。

那么回到ConnectInterceptor,它的作用就是为当前请求找到合适的连接，可能复用已有连接也可能是重新创建的连接，返回的连接由连接池负责决定。

##### 5.CallServerInterceptor

整个拦截器链中的最后一个拦截器，看一下源码。

关键方法intercept,如下：

```
@Override 
public Response intercept(Chain chain) throws IOException {
	    RealInterceptorChain realChain = (RealInterceptorChain) chain;
	    HttpCodec httpCodec = realChain.httpStream();
	    StreamAllocation streamAllocation = realChain.streamAllocation();
	    RealConnection connection = (RealConnection) realChain.connection();
	    Request request = realChain.request();
		
	    long sentRequestMillis = System.currentTimeMillis();
		 //1.首先写请求头
	    realChain.eventListener().requestHeadersStart(realChain.call());
	    httpCodec.writeRequestHeaders(request);
	    realChain.eventListener().requestHeadersEnd(realChain.call(), request);
	
	    Response.Builder responseBuilder = null;
	    if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
	      // If there's a "Expect: 100-continue" header on the request, wait for a "HTTP/1.1 100
	      // Continue" response before transmitting the request body. If we don't get that, return
	      // what we did get (such as a 4xx response) without ever transmitting the request body.
	      if ("100-continue".equalsIgnoreCase(request.header("Expect"))) {
	        httpCodec.flushRequest();
	        realChain.eventListener().responseHeadersStart(realChain.call());
	        responseBuilder = httpCodec.readResponseHeaders(true);
	      }
		   //2.然后写请求体
	      if (responseBuilder == null) {
	        // Write the request body if the "Expect: 100-continue" expectation was met.
	        realChain.eventListener().requestBodyStart(realChain.call());
	        long contentLength = request.body().contentLength();
	        CountingSink requestBodyOut =
	            new CountingSink(httpCodec.createRequestBody(request, contentLength));
	        BufferedSink bufferedRequestBody = Okio.buffer(requestBodyOut);
	
	        request.body().writeTo(bufferedRequestBody);
	        bufferedRequestBody.close();
	        realChain.eventListener()
	            .requestBodyEnd(realChain.call(), requestBodyOut.successfulCount);
	      } else if (!connection.isMultiplexed()) {
	        // If the "Expect: 100-continue" expectation wasn't met, prevent the HTTP/1 connection
	        // from being reused. Otherwise we're still obligated to transmit the request body to
	        // leave the connection in a consistent state.
	        streamAllocation.noNewStreams();
	      }
	    }
	
	    httpCodec.finishRequest();
		 //读取响应头
	    if (responseBuilder == null) {
	      realChain.eventListener().responseHeadersStart(realChain.call());
	      responseBuilder = httpCodec.readResponseHeaders(false);
	    }
	
	    Response response = responseBuilder
	        .request(request)
	        .handshake(streamAllocation.connection().handshake())
	        .sentRequestAtMillis(sentRequestMillis)
	        .receivedResponseAtMillis(System.currentTimeMillis())
	        .build();
	
	    realChain.eventListener()
	        .responseHeadersEnd(realChain.call(), response);
	    //判断响应码，读取响应体
	    int code = response.code();
	    if (forWebSocket && code == 101) {
	      // Connection is upgrading, but we need to ensure interceptors see a non-null response body.
	      response = response.newBuilder()
	          .body(Util.EMPTY_RESPONSE)
	          .build();
	    } else {
	      response = response.newBuilder()
	          .body(httpCodec.openResponseBody(response))
	          .build();
	    }
	
	    if ("close".equalsIgnoreCase(response.request().header("Connection"))
	        || "close".equalsIgnoreCase(response.header("Connection"))) {
	      streamAllocation.noNewStreams();
	    }
	
	    if ((code == 204 || code == 205) && response.body().contentLength() > 0) {
	      throw new ProtocolException(
	          "HTTP " + code + " had non-zero Content-Length: " + response.body().contentLength());
	    }
	
	    return response;
	  }
```

真个过程就是CallServerInterceptor向服务器发起真正的请求，并在接收服务器的返回后读取响应返回。


##### 最后

```
// Call the next interceptor in the chain.
		    RealInterceptorChain next = new RealInterceptorChain(
		        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
		    Interceptor interceptor = interceptors.get(index);
		    Response response = interceptor.intercept(next);
```	
	    
整个执行链就在拦截器与拦截器链中交替执行，最终完成所有拦截器的操作。



