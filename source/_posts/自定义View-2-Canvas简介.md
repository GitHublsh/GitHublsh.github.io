---
title: 自定义View(2)Canvas简介
date: 2017-09-21 10:56:35
tags: [自定义View]
---

### 一、Canvas简介


#### （1）Canvas简介

Canvas,即画布。

#### （2）Canvas详解

画布，就是我们用来绘制的载体，我们可以用画笔（Paint）在画布（Canvas）上绘制我们想要的图形。

* 首先介绍一下画笔Paint

	* 1.三个构造函数
		
		* Paint()	创建一个画笔
		* Paint(int flags) 创建一个画笔，指定flag。也可在不指定flag创建后，通过setFlag指定。例如：Paint.ANTI_ALIAS_FLAG(用于绘制时抗锯齿）
		* Paint(Paint paint)	使用已构造的Paint创建一个画笔

	* 2.常用方法
	
		* setAlpha(int a)	设置透明度
		* setARGB(int a, int r, int g, int b) 	设置画笔颜色，a-alpha透明度，r-red,g-green,b-blue
		* setAntiAlias(boolean aa)	设置抗锯齿
		* setColor(@ColorInt int color) 设置颜色
		* setDither(boolean dither) 设置抖动，设置后线条会相对柔和一些，不那么僵硬
		* setUnderlineText(boolean underlineText)	设置文本下划线
		* setStyle(Style style)	设置画笔风格。FILL内部填充，STROKE描边，FILL_AND_STROKE填充内部和描边
		* setStrikeThruText(boolean strikeThruText)	设置文本删除线
		* setFilterBitmap(boolean filter)		对bitmap进行滤波处理，true-去除优化，加快显示
		* setColorFilter(ColorFilter filter) 	设置颜色过滤
		
		······
		
	* 3.ColorFilter

		ColorFilter主要用来处理颜色.它有三个子类：ColorMatrixColorFilter、LightingColorFilter、PorterDuffColorFilter。下面一一进行解释。
		
		* ColorMatrixColorFilter,在Android中，图片是以一个个 RGBA 的像素点的形式加载到内存中的，所以如果需要改变图片的颜色，就需要针对这一个个像素点的RGBA的值进行修改。修改图片的RGBA值，需要ColorMatrix类支持.	
	
