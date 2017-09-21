---
title: 自定义View(一)View简介
date: 2017-08-26 22:06:02
tags: [自定义View]
---

#### 一、常见View继承关系

* 如下图
	
	![view](http://ot29getcp.bkt.clouddn.com/view%E7%BB%A7%E6%89%BF.png)
	
可以看到所有的控件都是最终继承自View。布局控件都是直接或间接继承自ViewGroup.

#### 二、自定义控件

* 大致可分为两大类：
	
	* 组合或继承基本控件TextView、ImageView等，加上自定义的内容。
	* 继承自View

	 
	
1. 组合控件或继承基本控件

	像这种自定义的控件，很好理解，就是在原有的基础上，自定义的进行组合或者添加新的功能属性等。自定义ViewGroup一般是利用现有的组件根据特定的布局方式来组成新的组件，大多继承自ViewGroup或Layout,包含子View
	
	举个简单的例子：ToolBar
	
	在Android 5.0推出的一个新的导航控件用于取代之前的ActionBar。如果去看ToolBar的源码，就可以发现，其实就是将几种基本控件组合在一起，实现开发者的更好的可定制性。
	
	看源码：
	
	![toolbar](http://ot29getcp.bkt.clouddn.com/images/toolbar.png)
	
	可以看出来ToolBar组合了ActionMenuView、TextView、ImageButton、ImageView等。相当于一个布局文件里面加了自己自定义的控件组合而成达到自己想要的效果。
	
	
2. 自定义View

 自定义View一般是没有需要的View,需要自己实现，继承自View、surfaceView或者其他的View,不包含子View

 * (1)屏幕坐标系

 	![zuobiaoxi](http://ot29getcp.bkt.clouddn.com/images/zuobiaoxi.png)
 	
 	手机屏幕的坐标系如图所示，是以手机屏幕左上顶角为坐标原点。
 * （2）View坐标系
 
 ![viewzuobiao](http://ot29getcp.bkt.clouddn.com/images/viewzuobiao.png)
 
 View坐标系都是相对父控件而言来确定位置的。
 	
 	* getTop():view顶部相对于父控件顶部距离
 	* getBottom():view底部相对于父控件顶部距离
 	* getLeft():view左边相对于父控件距离
 	* getRight():view右边相对于父控件距离
 	
 	
 	
3. View绘制流程

![viewhuizhi](http://ot29getcp.bkt.clouddn.com/images/customview.jpg)

 