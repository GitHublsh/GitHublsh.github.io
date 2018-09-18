---
title: Android四大组件之BroadCast
date: 2018-09-17 13:40:28
tags: [Android四大组件]
---

### Android四大组件之BroadCast

#### 一、广播概述

Android应用可以从Android系统和其他Android应用发送或接收广播消息，是观察者设计模式，即一对多的关系。例如，应用程序还可以发送自定义广播，以通知其他应用程序可能感兴趣的内容（例如，已下载了一些新数据）。

广播是一种广泛运用的在应用程序之间传输信息的机制。而BroadcastReceiver是对发送出来的广播进行过滤接收并响应的一类组件；

BroadcastReceiver自身并不实现图形用户界面，但是当它收到某个通知后，BroadcastReceiver可以启动Activity作为响应，或者通过NotificationMananger提醒用户，或者启动Service等等。

经常说”发送广播“和”接收广播“，表面上看广播作为Android广播机制中的实体，实际上这一实体本身是并不是以所谓的”广播“对象存在的，而是以”意图“（Intent）去表示。定义广播的定义过程，实际就是相应广播”意图“的定义过程，然后通过广播发送者将此”意图“发送出去。被相应的BroadcastReceiver接收后将会回调onReceive()函数。


#### 二、广播分类

Android中的BroadCast类型主要分为5类:

* Normal BroadCast(普通广播)
* System BroadCast(系统广播)
* Ordered BroadCast(有序广播)
* Sticky BroadCast(粘性广播)
* Local BroadCast(App应用内广播)


#### 三、广播的注册和接收

广播的注册分为两种：静态注册和动态注册

##### 1.静态注册

静态注册就是在AndroidManifest中注册，

	<receiver 
    //此广播接收者类是MyBroadCastReceiver
    android:name=".MyBroadCastReceiver" >
    //用于接收网络状态改变时发出的广播
    <intent-filter>
        <action android:name="com.test.mybroadcast" />
    </intent-filter>
	</receiver>
	
	
##### 2.动态注册

需要在代码中实现

    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction("com.action.myreceiver");
    myReceiver = new MyReceiver();
    registerReceiver(myReceiver, intentFilter);

注意：动态注册后记得解绑注册。


#### 四、常用广播的简单使用

##### 1.普通广播

普通广播的使用，在第三点中已举例说明，不再重复。

##### 2.系统广播

Android中内置了一些系统广播。常见的有开关机、网络、电话短信、拍照等等。

例如：

* 监听网络变化	android.net.conn.CONNECTIVITY_CHANGE

* 关闭或打开飞行模式	Intent.ACTION_AIRPLANE_MODE_CHANGED

* 充电时或电量发生变化	Intent.ACTION_BATTERY_CHANGED

* 电池电量低	Intent.ACTION_BATTERY_LOW

* 电池电量充足（即从电量低变化到饱满时会发出广播	Intent.ACTION_BATTERY_OKAY

* 系统启动完成后(仅广播一次)	Intent.ACTION_BOOT_COMPLETED

* 按下照相时的拍照按键(硬件按键)时	Intent.ACTION_CAMERA_BUTTON

* 屏幕锁屏	Intent.ACTION_CLOSE_SYSTEM_DIALOGS

* 设备当前设置被改变时(界面语言、设备方向等)	Intent.ACTION_CONFIGURATION_CHANGED

* 插入耳机时	Intent.ACTION_HEADSET_PLUG

* 插入外部储存装置（如SD卡）	Intent.ACTION_MEDIA_CHECKING

* 成功安装APK	Intent.ACTION_PACKAGE_ADDED

* 成功删除APK	Intent.ACTION_PACKAGE_REMOVED

* 重启设备	Intent.ACTION_REBOOT

* 屏幕被关闭	Intent.ACTION_SCREEN_OFF

* 屏幕被打开	Intent.ACTION_SCREEN_ON

* 关闭系统时	Intent.ACTION_SHUTDOWN

* 重启设备	Intent.ACTION_REBOOT


##### 3.有序广播

有序广播，也比较常用。发送广播和普通广播的区别是有序广播 通过sendOrderedBroadCast发送广播。
有序广播的顺序，体现在广播接收者中，是按照广播接收者的优先顺序来的，优先级在清单文件中广播接收者的intent-filter中通过设置priority属性来定义。
例如：
	
	<intent-filter android:priority="2000">
	
priority值越大，表示优先级越高

当优先级最高的广播接收者接收到广播后，可以通过setResult继续传递广播，也可以通过abortBroadcast()中断广播的继续传播

代码示例：

	<receiver
        android:name=".MyOrderedReceiver"
        android:enabled="true"
        android:exported="true">
        <intent-filter android:priority="1000">
            <action android:name="com.neil.ordered" />
        </intent-filter>
    </receiver>
    <receiver
        android:name=".MySecondOrderedReceiver"
        android:enabled="true"
        android:exported="true">
        <intent-filter android:priority="500">
            <action android:name="com.neil.ordered" />
        </intent-filter>
    </receiver>
    
   
 发送广播：
 
	void sendOrderedBroadcast (Intent intent, 
	                String receiverPermission)
	                
接收广播和普通广播一样，只是按照优先级有接收的先后顺序。

##### 4.粘性广播

注册与接收和普通广播是一样的，但是需要添加权限，否则会抛出异常。


	<uses-permission android:name="android.permission.BROADCAST_STICKY" />

和普通广播的区别是，sendStickyBroadcast它将发出的广播保存起来，一旦发现有人注册这条广播，则立即能接收到。比较简单，就不举例说明了。

##### 5.应用内广播

应用内广播更加安全。用法基本一样，只是应用内广播是通过LocalBroadcastManager来实现的。示例如下：

首先还是注册和解绑的操作，

	        LocalBroadcastManager.getInstance(this).registerReceiver(localBroadcastDemo,intentFilter);

	LocalBroadcastManager.getInstance(this).unregisterReceiver(localBroadcastDemo);


组装好Intent后，发送广播。

	LocalBroadcastManager.getInstance(MainActivity.this).sendBroadcast(intent);
	
广播接收者和普通广播一样。










	




