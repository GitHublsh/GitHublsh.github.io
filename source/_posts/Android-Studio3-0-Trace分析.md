---
title: Android Studio3.0+ Trace分析
date: 2018-11-07 11:04:26
tags: [Android性能优化]
---

#### 一、简要说明

在进行性能优化的时候，很多情况下需要去看方法耗时。这个时候我们就需要通过Trace来查看方法具体耗时情况。

#### 二、具体使用方法

##### 1.使用DDMS工具来查看trace进行分析

在Android Studio3.0 之前，在AS IDE面板中->Tools->Android Device Monitor 打开即可使用，通过点击Start Method Profiling按钮开始进行方法调用跟踪，停止时再点击停止，即可生成trace文件进行分析。

需要注意的是，在AS3.0+ DDMS从面板上移除了，官方给的说法是：

However, most components of the Android Device Monitor are deprecated in favor of updated tools available in Android Studio 3.0 and higher.

那就手动的去启动就OK了。（在Terminal中输入命令 monitor 启动DDMS）

##### 2.通过代码进行追踪

在需要分析的代码中添加

开始

	Debug.startMethodTracing()

结束

	Debug.stopMethodTracing()
	
	
