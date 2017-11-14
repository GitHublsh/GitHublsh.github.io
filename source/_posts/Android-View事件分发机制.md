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

		
2. 点击事件的分发中最重要的三个方法

	* dispatchTouchEvent(MotionEvent event)--事件分发

		* 用来进行事件分发，如果事件能够传递给当前View,那么这个方法一定会被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法影响，表示是否消耗了当前事件。
		
	* onInterceptTouchEvent(MotionEvent event)--事件拦截

		* 用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列中，此方法不会再次调用，返回结果表示是否拦截当前事件。
		
	* onTouchEvent(MotionEvent event)--事件消费

		* 在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一事件序列中，当前View无法再次接收到事件

3. 伪代码

		public boolean dispatchTouchEvent(MotionEvent ev){
			boolean touch = false;
			if(onInterceptTouchEvent(ev)){
				touch = onTouchEvent(ev);
			}else{
				touch = child.dispatchTouchEvent(ev);
			}
			return touch;
		}
     
   
  * 点击事件的传递规则：
  	
  		* 对于根ViewGroup来说，点击事件产生后，首先传递给它，这个时候它的dispatchTouchEvent就会被调用，	 
   