---
title: Android Studio3.0+ Trace分析
date: 2018-11-07 11:04:26
tags: [Android性能优化]
---


#### 一、简要说明

在进行性能优化的时候，很多情况下需要去看方法耗时。这个时候我们就需要通过Trace来查看方法具体耗时情况。

#### 二、具体使用方法

##### 1.使用DDMS工具来查看trace进行分析

在Android Studio3.0 之前，在AS IDE面板中->Tools->Android Device Monitor 打开即可使用，通过点击Start Method Profiling(为红色)按钮开始进行方法调用跟踪(点击后会变为黑色)，停止时再点击停止，即可生成trace文件进行分析。

需要注意的是，在AS3.0+ DDMS从面板上移除了，官方给的说法是：

However, most components of the Android Device Monitor are deprecated in favor of updated tools available in Android Studio 3.0 and higher.

那就手动的去启动就OK了。（在Terminal中输入命令 monitor 启动DDMS）

##### 2.通过代码进行追踪

在需要分析的代码中添加

开始

	Debug.startMethodTracing()

结束

	Debug.stopMethodTracing()
	
	
这样就会生成.trace文件了。

#### 三、分析trace文件

在面板中会有各种数据，这个时候就需要搞清楚各个参数所代表的具体含义了。


最关心的数据有：
很重要的指标：Calls + Recur Calls / Total , 最重要的指标： Cpu Time / Call
因为我们最关心的有两点，一是调用次数不多，但每次调用却需要花费很长时间的函数。这个可以从Cpu Time / Call反映出来。另外一个是那些自身占用时间不长，但调用却非常频繁的函数。这个可以从**Calls + Recur Calls / Total **反映出来。

当体验卡顿的时候，我们可以借助TraceView来定位问题
