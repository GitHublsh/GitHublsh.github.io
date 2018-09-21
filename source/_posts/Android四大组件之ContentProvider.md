---
title: Android四大组件之ContentProvider
date: 2018-09-18 15:20:49
tags: [Android四大组件]
---

### 一、简介

内容提供器（ContentProvider）主要用于在不同的应用程序之间实现数据共享的功能。它提供一整套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被访问数据的安全性。

内容提供器非常类似于数据库，可以使用insert()、update()、delete()、query()方法进行插入更新删除查询数据，大多数情况下，此数据存储在SqliteS数据库中。

