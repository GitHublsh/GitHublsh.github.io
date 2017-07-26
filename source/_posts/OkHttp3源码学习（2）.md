---
title: OkHttp3源码学习（2）
date: 2017-07-26 15:25:37
tags: [OkHttp3]
---

#### 上一节对OkHttp3做了一个简单的介绍及科普了一下使用。

那么，从这一节开始，进行源码分析解读···


#### 一、创建OkHttpClient对象
	
		OkHttpClient client = new OkHttpClient();
创建时，做了什么事情？

直接进OkHttpClient.java 

如果我们不做任何配置，那么就采用默认的配置，已经写好。

	public OkHttpClient() {
	    this(new Builder());
	  }
	  
	public Builder() {
      dispatcher = new Dispatcher();
      protocols = DEFAULT_PROTOCOLS;
      connectionSpecs = DEFAULT_CONNECTION_SPECS;
      eventListenerFactory = EventListener.factory(EventListener.NONE);
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
      pingInterval = 0;
    }
    
    
  当然，如果自己想设置一些参数:
  
	  OkHttpClient client = new OkHttpClient.Builder()  
	        .connectTimeout(10, TimeUnit.SECONDS)
	        .addInterceptor(...)
	        ....
	        .build();
	        
 个性化配置包你满意。
 
 
#### 二、创建网络请求


	Request request = new Request.Builder()  
        .addHeader("Connection", "Keep-Alive")
        .url("http://www.google.com")
        ....
        .build();
        
  这里默认发送的请求是GET：
  
	   public Builder() {
	      this.method = "GET";
	      this.headers = new Headers.Builder();
	    }
	    

发送POST请求，上一章节已经说明了请求方法，源码中实现：

	  public Builder post(RequestBody body) {
      return method("POST", body);
    }
    
    
#### 三、拿到Call对象

	Call call = client.newCall(request);
	
Call即是一个实际的访问请求，用户的每一个网络请求都是一个Call实例。

	/**
	 * A call is a request that has been prepared for execution. A call can be canceled. As this object
	 * represents a single request/response pair (stream), it cannot be executed twice.
	 */
	public interface Call extends Cloneable {
		···
	}
	
一个call就是一次已准备好的请求执行，并且可以被取消。这个请求对象是单个，所以不能执行两次。
        
	  /**
	   * Prepares the {@code request} to be executed at some point in the future.
	   */
	  @Override public Call newCall(Request request) {
	    return new RealCall(this, request, false /* for web socket */);
	  }
	  
实际在执行过程中，OkHttp会为每个请求创建一个RealCall.那么再进RealCall看看。


* 发起一个同步请求

		
		  @Override public Response execute() throws IOException {
		    synchronized (this) {
		      if (executed) throw new IllegalStateException("Already Executed");
		      executed = true;
		    }
		    captureCallStackTrace();
		    try {
		      client.dispatcher().executed(this);
		      Response result = getResponseWithInterceptorChain();
		      if (result == null) throw new IOException("Canceled");
		      return result;
		    } finally {
		      client.dispatcher().finished(this);
		    }
		  }
		  
		  
* 发起异步请求


		  @Override public void enqueue(Callback responseCallback) {
		    synchronized (this) {
		      if (executed) throw new IllegalStateException("Already Executed");
		      executed = true;
		    }
		    captureCallStackTrace();
		    client.dispatcher().enqueue(new AsyncCall(responseCallback));
		  }
		  
		  
	  
		synchronized void enqueue(AsyncCall call) {
		if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
		  runningAsyncCalls.add(call);
		  executorService().execute(call);
		} else {
		  readyAsyncCalls.add(call);
		}
		}
    

