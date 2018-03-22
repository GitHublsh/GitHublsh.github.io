---
title: IPC基础概念介绍
date: 2018-03-22 09:09:50
tags: [Android进阶]
---

#### 一、简介

* IPC是 Inter-Process Communication的缩写，意为进程间通信或跨进程通信，是指两个进程之间进行数据交换的过程。

* 线程是CPU调度的最小单元，同时线程是一种有限的系统资源。进程一般指一个执行单元，在PC和移动设备上指一个程序或者一个应用。一个进程可以包含多个线程，因此进程和线程是包含与被包含的关系。最简单的情况下，一个进程中只可以有一个线程，即主线程，在Android中也叫UI线程。


#### 二、Android中的多进程模式

 Android中开启多进程的方法：
 
	 <activity android:name=".BinderOneActivity"
	            android:process="com.neil.binder.one"/>
	 <activity android:name=".BinderTwoActivity"
	            android:process="com.neil.binder.two"/>
	 <activity android:name=".BinderThreeActivity"
	            android:process="com.neil.binder.three"/>
	            
在AndroidManifest.xml指定android:process属性，若没有指定process属性，默认的进程中，默认进程的进程名是包名。

日志输出：

	u0_a60    4080  1207  1255464 42888 ffffffff b74c94f5 S com.example.liushihan.glidedemo
	u0_a60    4113  1207  1256320 42124 ffffffff b74c94f5 S com.neil.binder.one
	u0_a60    4144  1207  1254256 42088 ffffffff b74c94f5 S com.neil.binder.two
	u0_a60    4170  1207  1254256 41744 ffffffff b74c94f5 S com.neil.binder.three

Android系统为每个应用分配一个唯一的UID，具有相同UID的应用才能共享数据。两个应用通过ShareUID跑在同一个进程中是有要求的，需要这两个应用有相同的ShareUID并且签名相同才可以。在这种情况下，它们可以互相访问对方的私有数据，比如data目录，组件信息等。


	            
#### 三、序列化和反序列化

1. 序列化

由于存在于内存中的对象都是暂时的，无法长期驻存，为了把对象的状态保持下来，这时需要把对象写入到磁盘或者其他介质中，这个过程就叫做序列化。

指把堆内存中的 Java 对象数据，通过某种方式把对象存储到磁盘文件中或者传递给其他网络节点（在网络上传输）。这个过程称为序列化。通俗来说就是将数据结构或对象转换成二进制串的过程

2. 反序列化

反序列化恰恰是序列化的反向操作，也就是说，把已存在在磁盘或者其他介质中的对象，反序列化（读取）到内存中，以便后续操作，而这个过程就叫做反序列化。

  概括性来说序列化是指将对象实例的状态存储到存储媒体（磁盘或者其他介质）的过程。在此过程中，先将对象的公共字段和私有字段以及类的名称（包括类所在的程序集）转换为字节流，然后再把字节流写入数据流。在随后对对象进行反序列化时，将创建出与原对象完全相同的副本。
  
  把磁盘文件中的对象数据或者把网络节点上的对象数据，恢复成Java对象模型的过程。也就是将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程