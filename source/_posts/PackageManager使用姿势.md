---
title: PackageManager使用姿势
date: 2017-10-25 20:04:03
tags: [PackageManager]
---

PackageManager它的主要职责是管理应用程序包。 通过PackageManager，我们就可以获取应用程序信息。

#### 一、主要功能

1. 安装、卸载应用
2. 查询权限
3. 查询Application相关信息

#### 二、常用方法

1）、public abstract PackageInfo getPackageInfo(String packageName, int flags)根据包名获取对应的PackageInfo，注意，此处的flags标签：
 GET_ACTIVITIES
GET_GIDS
GET_CONFIGURATIONS
GET_INSTRUMENTATION
GET_PERMISSIONS
GET_PROVIDERS
GET_RECEIVERS
GET_SERVICES
GET_SIGNATURES
GET_UNINSTALLED_PACKAGES
（2）、public abstract int[] getPackageGids(String packageName)，根据包名获取group-ids

（3）、public abstract PermissionInfo getPermissionInfo(String name, int flags)，根据包名和指定的flags获取指定的授权信息

（4）、public abstract List<PermissionGroupInfo> getAllPermissionGroups(int flags);获取所以PermissGroup集合

（5）、public abstract PermissionGroupInfo getPermissionGroupInfo(String name,
    int flags)根据指定的Group名称获取PermissionGroupInfo对象。

（6）、public abstract ApplicationInfo getApplicationInfo(String packageName,
            int flags)，根据指定的包名获取ApplicationInfo信息。

（7）、public abstract ActivityInfo getActivityInfo(ComponentName component,
            int flags)，根据指定的组件，获取ActivityInfo信息

（8）、public abstract ServiceInfo getServiceInfo(ComponentName component,
            int flags)，根据指定组件获取ServiceInfo

（9）、public abstract ProviderInfo getProviderInfo(ComponentName component,
            int flags)，根据指定组件名称获取ProviderInfo信息

（10）、public abstract List<PackageInfo> getInstalledPackages(int flags);获取所有安装的PackagInfo信息

（11）、public abstract List<PackageInfo> getPackagesHoldingPermissions(
            String[] permissions, int flags);获取具有特定权限的PackagInfo

（12）、public abstract List<ApplicationInfo> getInstalledApplications(int flags);获取安装的ApplicationInfo信息

（13）、public abstract boolean addPermission(PermissionInfo info);添加权限

（14）、public abstract void removePermission(String name);移除权限

2、PackageInfo用于描述mainfest中所有描述信息。    
常见字段：   
（1）、public String packageName;包名    
（2）、public String[] splitNames;   
（3）、public int versionCode;版本号　　　　　
（4）、public String versionName;版本名称    
（5）、public ApplicationInfo applicationInfo;    
（6）、public long firstInstallTime;第一次安装时间   
（7）、public long lastUpdateTime;上次更新时间     
（8）、public ActivityInfo[] activities;所有的Activity信息     
（9）、public ActivityInfo[] receivers; 所有的广播接收者    
（10）、public ServiceInfo[] services;所有的服务信息     
（11）、public ProviderInfo[] providers;获取ContentProvide     
（12）、public PermissionInfo[] permissions;所有的权限信息


#### 三、AndroidManifest文件结构

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
	
	


#### 四、获取信息

1. ApplicationInfo类测试：获取Application节点信息

 * 示例：

			ApplicationInfo applicationInfo = getApplicationInfo();  
			 Log.d("lsh",applicationInfo.className + "\n" +  
			        applicationInfo.dataDir+"\n" +  
			        applicationInfo.permission + "\n"  
			        + applicationInfo.packageName + "\n"  
			        + applicationInfo.processName + "\n"  
			        + applicationInfo.taskAffinity + "\n"  
			);  
	

 * Logcat:

			3402-3402/com.example.liushihan.glidedemo D/lsh: null                                                 
			    /data/data/com.example.liushihan.glidedemo
			    null
			 	com.example.liushihan.glidedemo
			 	com.example.liushihan.glidedemo
			 	com.example.liushihan.glidedemo
			 	
2. 获取所有安装的Packages

 * 示例：

			List<PackageInfo> listPack = getPackageManager().getInstalledPackages(PackageManager.GET_ACTIVITIES);


 * Logcat:
   部分日志

			 D/lsh: PackageInfo{132b54a7 com.android.smoketest}
			com.example.liushihan.glidedemo D/lsh: PackageInfo{3d681d54 com.example.android.livecubes}
			com.example.liushihan.glidedemo D/lsh: PackageInfo{213c44fd com.android.providers.telephony}
			
			
			
3. 获取指定应用的PackageInfo

 * 示例：

			PackageManager packageManager = getPackageManager();
			        PackageInfo packageInfo = null;
			        try {
			            packageInfo = packageManager.getPackageInfo("com.example.liushihan",
			                    PackageManager.GET_ACTIVITIES);
			            Log.d("lsh", packageInfo.packageName + "\n"
			                    + packageInfo.versionName + "\n"
			            );
			        } catch (PackageManager.NameNotFoundException e) {
			            e.printStackTrace();
			        }
			        
			        
 * Logcat

			 D/lsh: com.example.liushihan.glidedemo
			                             1.0
			                             
			                             
			       
4. 获取应用程序中的permission

 * 示例：

			   try {
			            PermissionInfo permissionInfo = getPackageManager().getPermissionInfo("android.permission.INTERNET",
			                    PermissionInfo.PROTECTION_NORMAL);
			            List<PermissionGroupInfo> list = getPackageManager().getAllPermissionGroups(PackageManager.PERMISSION_GRANTED);
			            Log.d("lsh",
			                    permissionInfo.group + "\n"
			                            + permissionInfo.packageName + "\n"
			                            + permissionInfo.name + "\n"
			                            + permissionInfo.flags + "\n"
			            );
			        } catch (PackageManager.NameNotFoundException e) {
			            e.printStackTrace();
			        }
			        
			        
   * Logcat:

		    D/lsh: android.permission-group.NETWORK
		                                      android
		                                      android.permission.INTERNET
		                                      0
		                                      
                  
                  
5. 获取应用程序中执行的ActivityInfo

* 示例：


		ComponentName componentName = new ComponentName("com.example.liushihan.glidedemo","com.example.liushihan.glidedemo.MainActivity");
		        try {
		            @SuppressLint("WrongConstant") ActivityInfo activityInfo = getPackageManager().getActivityInfo(componentName, PackageManager.GET_ACTIVITIES);
		            Log.d("lsh:activityInfo",activityInfo.name + "\n"
		                    + activityInfo.packageName +"\n"
		                    + activityInfo.targetActivity
		            );
		        } catch (PackageManager.NameNotFoundException e) {
		            e.printStackTrace();
		        }  
		        
		        
* Logcat:

		
		4829-4829/com.example.liushihan.glidedemo D/lsh:activityInfo: com.example.liushihan.glidedemo.MainActivity
		                                                                                 com.example.liushihan.glidedemo
		                                                                                 null
		                                                                                 
		                                                                                 