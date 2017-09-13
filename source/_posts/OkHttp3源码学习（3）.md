---
title: OkHttp3源码学习(3)-拦截器链详解
date: 2017-07-27 17:02:28
tags: [OkHttp3]
---
#### 发起请求

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
        
* 发起请求时：client.newCall(request)。

		@Override public Call newCall(Request request) {
	    return new RealCall(this, request, false /* for web socket */);
	  	}
  
  实际上就是创建一个RealCall的实例。
  
*  然后call.enqueue,源码实现就是将RealCall加到任务队列中，等合适的机会去执行。

		@Override public void enqueue(Callback responseCallback) {
	    synchronized (this) {
	      if (executed) throw new IllegalStateException("Already Executed");
	      executed = true;
	    }
	    captureCallStackTrace();
	    client.dispatcher().enqueue(new AsyncCall(responseCallback));
	  	}

#### AsyncCall

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
  
  
  AsyncCall会执行execute方法。execute方法逻辑很简单：
  

* 通过调用getResponseWithInterceptorChain获取服务器返回结果，失败或者成功

		Response response = getResponseWithInterceptorChain();
  

* 通知任务分发器该任务结束

  		client.dispatcher().finished(this);


        
#### 构建拦截器链

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


从源码来看，基本逻辑就是：

* 创建一系列拦截器，加到拦截器数组中。
* 创建拦截器链RealInterceptorChain
* 执行拦截器链中的proceed方法



#### RealInterceptorChain

	/**
	 * A concrete interceptor chain that carries the entire interceptor chain: all application
	 * interceptors, the OkHttp core, all network interceptors, and finally the network caller.
	 */
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


可以看到procees方法的逻辑：
* 创建下一个拦截链（代码中的next），传入index+1，使创建的下一个拦截器链只能从下一个拦截器访问。

	// Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
        
* 获取索引为index的interceptor,执行索引为index的intercept方法。
	
		Interceptor interceptor = interceptors.get(index);
    	Response response = interceptor.intercept(next);


#### RetryAndFollowUpInterceptor


	
	  @Override public Response intercept(Chain chain) throws IOException {
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
	
      if (!sameConnection(response, followUp.url())) {
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
	
	
	
* 发起请求前拦截器对request处理
* 然后调用下一个拦截器，获取Response

调用的关键：

	try {
	        response = ((RealInterceptorChain) chain).proceed(request, streamAllocation, null, null);
	        releaseConnection = false;
	      }

那么这个时候就会去调用下一个拦截器。对response进行处理，返回给上一个拦截器


		 // Call the next interceptor in the chain.
		    RealInterceptorChain next = new RealInterceptorChain(
		        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
		    Interceptor interceptor = interceptors.get(index);
		    Response response = interceptor.intercept(next);
		    
		    
整个执行链就在拦截器与拦截器链中交替执行，最终完成所有拦截器的操作。




