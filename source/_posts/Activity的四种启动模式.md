---
title: Activity的四种启动模式
date: 2018-04-15 14:33:26
tags: [Activity]
---

作为基础回顾~


#### 一、Activity的四种启动模式简介

Activity是通过Activity栈来进行管理的。当一个Activity启动时，那么就会根据系统配置来将Activity压入栈中。当Back返回或销毁时,Activity出栈。

如果不指定Activity的启动模式，那么将以默认模式启动。如果要指定启动模式，在AndroidManifest中指定Activity的启动模式。例如：

	<activity android:name="com.app.TestActivity" android:launchMode="standard">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
	</activity>

#### 二、四种模式详解


1. standard

standard,标准模式。是Activity的系统默认启动模式。每次启动一个Activity都会创建一个新的实例，不管该Activity的实例是否存在。

2. singletop

singletop,栈顶复用模式。这种模式下，如果启动的Activity就在栈顶，那么复用栈顶的Activity，并且调用onNewIntent()方法。通过这个方法，可以取到当前请求的数据。如果启动的Activity已存在，但是不在栈顶，那么就会重新创建新的实例。

3. singletask

singletask,栈内复用模式。在这种模式下，是单实例模式，在这种模式下，只要启动的Activity在任何一个任务栈中存在，就不会重新创建新的实例。和singletop一样，会调用onNewIntent方法。

那么在启动的时候，先找有没有实例在Activity栈中，如果有那么就会将要启动的Activity置于栈顶，并且调用onNewIntent方法。如果没有那么就会创建新的实例压入栈中。

需要注意的是，singletask默认是cleartop,会将栈内将要启动的Activity实例上面的Activity全部出栈。使启动的Activity实例位于栈顶。

4. singleInstance

singleInstance,单个实例模式。这种模式下，Activity只能单独的在一个任务栈中存在。当启动一个Activity启动模式为singleInstance时，系统会新建一个任务栈，该实例压入栈中，仅有该实例存在。