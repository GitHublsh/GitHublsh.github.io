---
title: OkHttp3源码学习（1）
date: 2017-07-25 10:19:43
tags: [OkHttp3]
---


### 一、基本使用

基本步骤：

* 创建OkHttpClient对象
 
  OkHttpClient client = new OkHttpClient();
	
* 创建网络请求

  Request request = new Request.Builder().url(url).build();
  
* 发送请求

  Call call = client.newCall(request).excute();(或者异步)
	
	


1. GET请求
	* 同步
			
			OkHttpClient client = new OkHttpClient();
			
			String run(String url) throws IOException {
			Request request = new Request.Builder()
				.url(url)
				.build();
			
			Response response = client.newCall(request).execute();
			return response.body().string();
	}
			
	* 异步
	
			OkHttpClient client = new OkHttpClient();
			Request request = new Request.Builder()
				.url(url)
				.build();
			client.newCall(request).enqueue(new CallBack(){
				@Override
				public void onFailure(Request request,IOException e){
				}
				@Override
				public void onResponse(Response response){
				}
			})
			
			
2. POST请求
	* 同步请求
	
			public static final MediaType JSON
			    = MediaType.parse("application/json; charset=utf-8");
			
			OkHttpClient client = new OkHttpClient();
			
			String post(String url, String json) throws IOException {
			  RequestBody body = RequestBody.create(JSON, json);
			  Request request = new Request.Builder()
			      .url(url)
			      .post(body)
			      .build();
			  Response response = client.newCall(request).execute();
			  return response.body().string();
			}
			
	* 异步请求
			
			public static final MediaType JSON
			    = MediaType.parse("application/json; charset=utf-8");
			
			OkHttpClient client = new OkHttpClient();
			
			  RequestBody body = RequestBody.create(JSON, json);
			  Request request = new Request.Builder()
			      .url(url)
			      .post(body)
			      .build();
			  client.newCall(request).enqueue(new CallBack(){
			  		@Override
					public void onFailure(Request request,IOException e){
					}
					@Override
					public void onResponse(Response response){
					}
			  })
			  
			  
			  
#### 二、架构总览
借用网上的图···（侵权必删）
![“okhttp3整体架构”](http://ot29getcp.bkt.clouddn.com/images/okhttp3all.png)


#### 三、OkHttp的优点

网上的各路大神已经总结过很多遍了，我再来一遍，加深记忆···

* 支持HTTP2/SPDY黑科技
* socket自动选择最好路线，并支持自动重连
* 拥有自动维护的socket连接池，减少握手次数
* 拥有队列线程池，轻松写并发
* 拥有Interceptors轻松处理请求与响应（比如透明GZIP压缩,LOGGING）
* 实现基于Headers的缓存策略


#### 四、小结

首先对OkHttp有一个整体的认识，了解基本用法。熟悉整体框架结构。下篇开始进行源码的解读···



		
	   
	   