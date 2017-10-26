---
title: PackageManager使用姿势
date: 2017-10-25 20:04:03
tags: [PackageManager]
---

首先看一下AndroidManifest.xml文件结构。

例如：

	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	    package="com.example.test">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".TestActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service
        	android:name="testservice"
        	android:enable="true"
        	android:exproted="true">
        	</service>
    </application>

	</manifest>
	
	
	
	
根据AndroidManifest的各个节点可以看出来其中的对应关系：
	
	* 一个Package对应一个Application
	* 一个Application对应n个Activity、Service等等

* 通过包名判断APP是否安装
* 通过遍历获取AndroidManifest.xml文件信息
	
	

