---
title: Android四大组件之ContentProvider
date: 2018-09-18 15:20:49
tags: [Android四大组件]
---

### 一、简介

ContentProvider为不同的软件之间数据共享，提供统一的接口

内容提供器（ContentProvider）主要用于在不同的应用程序之间实现数据共享的功能。它提供一整套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被访问数据的安全性。

内容提供器非常类似于数据库，可以使用insert()、update()、delete()、query()方法进行插入更新删除查询数据，大多数情况下，此数据存储在SqliteS数据库中。

不过和Sqlite不同的是，内容提供器的几种增删改查的方法是不接受表名参数的，而是使用Uri。

### 二、内容Uri

ContentProvider的内容Uri格式是固定的，下面详细介绍一下。

	content://<authority>/<data_path>
	
	例如：content://com.neil.myprovider/table_contact/666
	
* content://是通用前缀，表示该Uri用于ContentProvider定位资源。

*  authority：授权信息，以区别不同的ContentProvider.

*  path:表名，用来区分ContentProvider中的不同的表

### 三、基本使用

内容提供者是android应用程序的基本构建块之一，它们封装数据并将封装的数据通过单一的ContentResolver接口提供给应用程序。当你需要在多个应用之间共享数据的时候就需要用到内容提供者。例如，手机中的联系人数据会被多个应用所用到所以必须要用内容提供者存储起来。如果你不需要在多个应用之间共享数据，你可以使用一个数据库，直接通过SQLite数据库。 当通过contentresolver发送一个请求时，系统会检查给定的URI并把请求传给有注册授权的Contentprovider。 UriMatcher类有助于解析uri。

需要实现的方法有：

	public boolean onCreate() 在创建ContentProvider时调用
	
	public Cursor query(Uri, String[], String, String[], String) 用于查询指定Uri的ContentProvider，返回一个Cursor
	
	public Uri insert(Uri, ContentValues) 用于添加数据到指定Uri的ContentProvider中，(外部应用向ContentProvider中添加数据)
	
	public int update(Uri, ContentValues, String, String[]) 用于更新指定Uri的ContentProvider中的数据
	
	public int delete(Uri, String, String[]) 用于从指定Uri的ContentProvider中删除数据
	
	public String getType(Uri) 用于返回指定的Uri中的数据的MIME类型
	


需要在AndroidManifest.xml中进行声明。
