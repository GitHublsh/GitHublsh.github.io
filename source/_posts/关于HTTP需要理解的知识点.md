---
title: 关于HTTP需要理解的知识点
date: 2016-11-13 15:24:05
tags: [HTTP]
---


## 关于HTTP需要知道的知识点

### HTTP简介

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。

HTTP是一个基于TCP/IP通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。

### HTTP消息结构

#### 客户端请求消息

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成，下图给出了请求报文的一般格式。

![“请求报文”](http://ot29getcp.bkt.clouddn.com
/images/request.png) 


#### 服务器相应消息

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

![“服务器响应”](http://ot29getcp.bkt.clouddn.com
/images/httpmessage.jpg) 


#### 实例

使用GET来传递数据的实例：

客户端请求：

	GET /hello.txt HTTP/1.1
	User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
	Host: www.example.com
	Accept-Language: en, mi
	
服务端响应：

	HTTP/1.1 200 OK
	Date: Mon, 27 Jul 2009 12:28:53 GMT
	Server: Apache
	Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
	ETag: "34aa387-d-1568eb00"
	Accept-Ranges: bytes
	Content-Length: 51
	Vary: Accept-Encoding
	Content-Type: text/plain
	
输出结果：

	Hello World! My payload includes a trailing CRLF.
	
### HTTP请求方法

* HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。

* HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。

序号 | 方法 | 描述 
---- |---- |----
1|GET|请求指定的页面信息，并返回实体主体。
2|HEAD|类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
3|POST|向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。
4|PUT|从客户端向服务器传送的数据取代指定的文档的内容。
5|DELETE|请求服务器删除指定的页面。
6|CONNECT|HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。
7|OPTIONS|允许客户端查看服务器的性能。
8|TRACE|回显服务器收到的请求，主要用于测试或诊断。


### HTTP响应头信息

HTTP请求头提供了关于请求，响应或者其他的发送实体的信息。

应答头 | 说明
---- |----
Allow | 服务器支持哪些请求方法（如GET、POST等）
Content-Encoding	|文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader("Accept-Encoding")）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。
Content-Length|表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入 ByteArrayOutputStream，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo(response.getOutputStream()发送内容。
Content-Type|表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。
Date|当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。
Expires|应该在什么时候认为文档已经过期，从而不再缓存它
Last-Modified	|文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置。
Location|表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。
Refresh|表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader("Refresh", "5; URL=http://host/path")让浏览器读取指定的页面。注意这种功能通常是通过设置HTML页面HEAD区的＜META HTTP-EQUIV="Refresh" CONTENT="5;URL=http://host/path"＞实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。注意Refresh的意义是"N秒之后刷新本页面或访问指定页面"，而不是"每隔N秒刷新本页面或访问指定页面"。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是＜META HTTP-EQUIV="Refresh" ...＞。注意Refresh头不属于HTTP 1.1正式规范的一部分，而是一个扩展，但Netscape和IE都支持它。
Server|服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。
Set-Cookie|设置和页面关联的Cookie。Servlet不应使用response.setHeader("Set-Cookie", ...)，而是应使用HttpServletResponse提供的专用方法addCookie。参见下文有关Cookie设置的讨论。
WWW-Authenticate|客户应该在Authorization头中提供什么类型的授权信息？在包含401（Unauthorized）状态行的应答中这个头是必需的。例如，response.setHeader("WWW-Authenticate", "BASIC realm=＼"executives＼"")。注意Servlet一般不进行这方面的处理，而是让Web服务器的专门机制来控制受密码保护页面的访问（例如.htaccess）。

### HTTP状态码
常见的HTTP状态码：

* 200 - 请求成功
* 301 - 资源（网页等）被永久转移到其它URL
* 404 - 请求的资源（网页等）不存在
* 500 - 内部服务器错误
