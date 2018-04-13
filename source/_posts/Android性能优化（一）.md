---
title: Android性能优化（一）布局优化
date: 2018-04-13 14:57:40
tags: Android性能优化
---

布局优化主要的思想就一点：减少布局的层级

常用的优化方式有三种：

#### 一、\<include\>标签

\<include\>标签可以将制定的布局加载到当前布局中。可以多处引用，类似于工具类的公共方法。

举个例子：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="vertical"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <include layout="@layout/include_comm_new_topbar_header"/>
	
	    <android.support.v7.widget.RecyclerView
	        android:id="@+id/courier_recycle_change_record"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent" />
	</LinearLayout>
	
引用的部分，
	
	<?xml version="1.0" encoding="utf-8"?>
	
	<ComTopBarNew xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/toolBar"
	    android:layout_width="match_parent"
	    android:layout_height="@dimen/com_toolbar_height"
	    android:background="@color/comm_color18"
	    />
	    
需要注意的是，include标签仅支持layout开头的属性；当在include的布局中指定了id,在当前的布局文件中也指定了id,那么是以include中指定的id为准的。


#### 二、\<merge\>标签

\<merge\>标签，一般是和<include>标签配合使用的，举个例子：


	<?xml version="1.0" encoding="utf-8"?>
	<merge>
	
	    <View style="@style/Comm.Divider.Horizontal" />
	</merge>
	
	
使用场景也很好理解，如果当前布局是LinearLayout，被包含的布局也是LinearLayout,那么就可以使用<merge>标签，这样做就减少了布局的层级了。

#### 三、 \<ViewStub\>标签

\<ViewStub\>标签，ViewStub继承自View,标签最大的特点就是当你需要的时候才会加载，但并不会影响UI初始化的性能。各种不常用的布局文件如进度条、显示错误信息等可以使用<ViewStub />标签以减少内存使用量，加快渲染速度。<ViewStub />标签是一个不可见的，大小为0的View。所以不会参与任何布局和绘制。

在需要使用的时候再去加载显示，这样就提高了程序初始的性能了。

举个例子：

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
	    xmlns:tools="http://schemas.android.com/tools"  
	    android:layout_width="match_parent"  
	    android:layout_height="match_parent"  
	    android:orientation="vertical"  
	    tools:context=".MainActivity">  
	  
	    <Button  
	        android:id="@+id/btn_show"  
	        android:layout_width="wrap_content"  
	        android:layout_height="wrap_content"  
	        android:text="Show"/>  
	    <Button  
	        android:id="@+id/btn_gone"  
	        android:layout_width="wrap_content"  
	        android:layout_height="wrap_content"  
	        android:text="GONE"/>  
	  
	    <LinearLayout  
	        android:layout_width="wrap_content"  
	        android:layout_height="wrap_content">  
	    <ViewStub  
	        android:id="@+id/viewstub"  
	        android:layout_width="wrap_content"  
	        android:layout_height="wrap_content"  
	        android:layout="@layout/view"/>  
	    </LinearLayout>  
	  
	</LinearLayout> 

在需要展示的时候，在设置显示出来，即按需加载。ViewStub是一个轻量级的View，它一个看不见的，不占布局位置，占用资源非常小的控件。可以为ViewStub指定一个布局，在Inflate布局的时候，只有ViewStub会被初始化，然后当ViewStub被设置为可见的时候，或是调用了ViewStub.inflate()的时候，ViewStub所向的布局就会被Inflate和实例化，然后ViewStub的布局属性都会传给它所指向的布局。这样，就可以使用ViewStub来方便的在运行时，要还是不要显示某个布局。

那么如何加载？

两种方式：

 * stub.setVisibility(View.VISIBLE); 

 * stub.inflate();