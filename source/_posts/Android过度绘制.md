---
title: Android过度绘制
date: 2018-07-27 11:17:23
tags: [Android]
---

#### 一、基础知识

android 的渲染主要分为两个组件：1.CPU 2.GPU，由这两者共同完成在屏幕上绘制 。 

* CPU：中央处理器,它集成了运算,缓冲,控制等单元,包括绘图功能.CPU将对象处理为多维图形,纹理(Bitmaps、Drawables等都是一起打包到统一的纹理)。 


* GPU：一个类似于CPU的专门用来处理Graphics的处理器,用来帮助加快格栅化操作,当然,也有相应的缓存数据(例如缓存已经光栅化过的bitmap等)机制。 

* OpenGL ES：手持嵌入式设备的3DAPI,跨平台的、功能完善的2D和3D图形应用程序接口API,有一套固定渲染管线流程。 

* DisplayList：把XML布局文件转换成GPU能够识别并绘制的对象。这个操作是在DisplayList的帮助下完成的。DisplayList持有所有将要交给GPU绘制到屏幕上的数据信息。 

* 栅格化：将图片等矢量资源,转化为一格格像素点的像素图,显示到屏幕上。 

* 垂直同步VSYNC：让显卡的运算和显示器刷新率一致以稳定输出的画面质量。它告知GPU在载入新帧之前，要等待屏幕绘制完成前一帧。下面的三张图分别是GPU和硬件同步所发生的情况,Refresh Rate:屏幕一秒内刷新屏幕的次数,由硬件决定,例如60Hz.而Frame Rate:GPU一秒绘制操作的帧数,单位fps。
#### 二、什么是过度绘制？

过度绘制（Overdraw）描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次重叠的 UI 结构里面，如果不可见的 UI 也在做绘制的操作，会导致某些像素区域被绘制了多次，同时也会浪费大量的 CPU 以及 GPU 资源，绘制示意图如下图所示

![overdraw](http://ot29getcp.bkt.clouddn.com//blogoverdraw.png)


* 原色：没有过度绘制 

* 蓝色：1 次过度绘制 

* 绿色：2 次过度绘制 

* 粉色：3 次过度绘制 

* 红色：4 次及以上过度绘制


#### 三、过度绘制优化

##### 1. 去除Activity自带的默认window背景

一般应用默认继承的主题都会有一个默认的 windowBackground ，比如默认的 Light 主题：

	<style name="Theme.Light"> 
	<item name="isLightTheme">true</item> 
	<item name="windowBackground">@drawable/screen_background_selector_light</item>
	 ...
	  </style>
	  
但是一般界面都会自己设置界面的背景颜色或者列表页则由 item 的背景来决定，所以默认的 Window 背景基本用不上，如果不移除就会导致所有界面都多 1 次绘制。

可以在应用的主题中添加如下的一行属性来移除默认的 Window 背景：

	<item name="android:windowBackground">@android:color/transparent</item>
	 <!-- 或者 --> 
	 <item name="android:windowBackground">@null</item>
	 
	 
或者在 BaseActivity 的 onCreate() 方法中使用下面的代码移除：

	 getWindow().setBackgroundDrawable(null); 
	 或者
	 getWindow().setBackgroundDrawableResource(android.R.color.transparent);



移除默认的 Window 背景的工作在项目初期做最好，因为有可能有的界面未设置背景色，这就会导致该界面显示成黑色的背景，如下所示，如果是后期移除的，就需要检查移除默认 Window 背景之后的界面是否显示正常。


##### 2.移除不必要的背景

还是上面的那个界面，因为移除了默认的 Window 背景，所以在布局中设置背景为白色：