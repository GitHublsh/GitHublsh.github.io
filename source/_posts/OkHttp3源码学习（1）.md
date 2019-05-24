---
title: OkHttp3源码学习（1）-简单实用教程
date: 2017-07-25 10:19:43
tags: [OkHttp3]
---

背景简介：

尽管Google在大部分安卓版本中推荐使用HttpURLConnection，但是这个类相比HttpClient实在是太难用，太弱爆了。
OkHttp是一个相对成熟的解决方案，据说Android4.4的源码中可以看到HttpURLConnection已经替换成OkHttp实现了。所以我们更有理由相信OkHttp的强大。

OkHttp 处理了很多网络疑难杂症：会从很多常用的连接问题中自动恢复。如果您的服务器配置了多个IP地址，当第一个IP连接失败的时候，OkHttp会自动尝试下一个IP。OkHttp还处理了代理服务器问题和SSL握手失败问题。

使用 OkHttp 无需重写程序中的网络代码。OkHttp实现了几乎和java.net.HttpURLConnection一样的API。如果你用了 Apache HttpClient，则OkHttp也提供了一个对应的okhttp-apache 模块。

简单来说，其他的太难用了，这个才是最好用的，不用你会后悔的~



### 一、基本使用

基本步骤：

* 创建OkHttpClient对象
 
  OkHttpClient client = new OkHttpClient();
	
* 创建网络请求

  Request request = new Request.Builder().url(url).build();
  
* 发送请求,得到返回

  Response response = client.newCall(request).excute();(或者异步)
	
	


1. GET请求


```
//同步请求
OkHttpClient client = new OkHttpClient();
	
String run(String url) throws IOException {
Request request = new Request.Builder()
	.url(url)
	.build();
	
Response response = client.newCall(request).execute();
return response.body().string();
}
	
```
		
```
//异步请求
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
```
			
2. POST请求

	
```
//同步请求
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
```	

```
//异步请求
public static final MediaType JSON= MediaType.parse("application/json; charset=utf-8");	
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
```
		  
#### 二、架构总览
借用网上的图···（侵权必删）

![整体架构](https://github.com/GitHublsh/BlogPic/raw/master/okhttp%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84.jpg)

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



		
	   
	   