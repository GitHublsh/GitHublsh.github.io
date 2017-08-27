---
title: 自定义View(一)
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

	 
	
