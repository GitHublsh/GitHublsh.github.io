---
title: Android 7.0 so库兼容问题
date: 2018-05-18 10:38:07
tags: [Android]
---

目前做的项目中，是用sqlcipher来进行数据库加密的。在适配的过程中遇到了问题。

系统为Android7.0时，在进入项目APP的时候，系统会弹出一个对话框。如图，

![加密库兼容问题](http://ot29getcp.bkt.clouddn.com//blog/Screenshot_20180525-155021.png)

发扬面向百度编程的精神，百度到网上相关资料分析。

问题分析，引入的sqlicipher.so库版本太低了，项目中引用的是3.1.0的版本。Android7.0中会报问题，系统就会弹框提示。

解决方案：

可以采用gradle引用更高版本的包，

compile ‘net.zetetic:android-database-sqlcipher:3.5.9@aar’ 
	
或者

compile ‘net.zetetic:android-database-sqlcipher:3.5.9’ 


对比低版本的包。发现低版本中没有X86_64,可能问题出在这里。

详见：https://blog.csdn.net/qklnmc/article/details/77967636

