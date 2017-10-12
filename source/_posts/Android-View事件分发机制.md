---
title: Android View事件分发机制
date: 2017-10-11 17:27:08
tags: [View事件分发机制]
---

### 一、点击事件的传递规则

   首先，要明白点击事件的分发就是MotionEvent事件的分发过程。
   
   1. MotionEvent
   
      常见的动作：
      
      常见的动作常量：
      
      * public static final int ACTION_DOWN = 0;单点触摸动作
    
      * public static final int ACTION_UP = 1;单点触摸离开动作
      * public static final int ACTION_MOVE = 2;触摸点移动动作
      * public static final int ACTION_CANCEL = 3;触摸动作取消
      * public static final int ACTION_OUTSIDE = 4;触摸动作超出边界
      * public static final int ACTION_POINTER_DOWN = 5;多点触摸动作
      * public static final int ACTION_POINTER_UP       = 6;多点离开动作


		主要就是：
		
		* ACTION_DOWN--手指刚接触屏幕
		* ACTION_MOVE--手指在屏幕上滑动
		* ACTION_UP--手指离开屏幕
     
   
   