---
title: 阿里百川Hotfix 1.4.0 Android接入
date: 2017-07-12 17:33:45
tags: [Android]
---




### 一.Android Studio签名打包
自行百度。

### 二.基本配置依赖
maven仓库地址：

	repositories {
	   maven {
	       url "http://repo.baichuan-android.taobao.com/content/groups/BaichuanRepositories"
	   }
	}
gradle坐标版本依赖：

	dependencies {
	    compile 'com.taobao.android:alisdk-hotfix:1.4.0'
	}

### 三.配置权限
HotFix SDK使用到以下权限

	<! -- 网络权限 -->
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	<! -- 外部存储读权限 -->
	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

### 四.配置AndroidManifest文件

在AndroidManifest.xml中间的application节点下添加如下配置：

	<meta-data
	android:name="com.taobao.android.hotfix.APPSECRET"
	android:value="your-app-secret" />
	<meta-data
	android:name="com.taobao.android.hotfix.RSASECRET"
	android:value="your-rsa-secret" />
app-secret为阿里百川开发者申请百川平台HotFix服务申请得到的App Secret和RSA密钥

### 五.接入SDK
 接入范例

initialize的调用应该尽可能的早. 强烈推荐在Application.onCreate()中进行SDk初始化以及查询服务器是否有可用补丁的操作.

	HotFixManager.getInstance().setContext(this)
	                .setAppVersion(appVersion)
	                .setAppId(appId)
	                .setAesKey(null)
	                .setSupportHotpatch(true)
	                .setEnableDebug(true)
	                .setPatchLoadStatusStub(new PatchLoadStatusListener() {
	                    @Override
	                    public void onload(final int mode, final int code, final String info, final int handlePatchVersion) {
	                        // 补丁加载回调通知
	                        if (code == PatchStatusCode.CODE_SUCCESS_LOAD) {
	                            // TODO: 10/24/16 表明补丁加载成功
	                        } else if (code == PatchStatusCode.CODE_ERROR_NEEDRESTART) {
	                            // TODO: 10/24/16 表明新补丁生效需要重启. 业务方可自行实现逻辑, 提示用户或者强制重启, 建议: 用户可以监听进入后台事件, 然后应用自杀
	                        } else if (code == PatchStatusCode.CODE_ERROR_INNERENGINEFAIL) {
	                            // 内部引擎加载异常, 推荐此时清空本地补丁, 但是不清空本地版本号, 防止失败补丁重复加载
	                            //HotFixManager.getInstance().cleanPatches(false);
	                        } else {
	                            // TODO: 10/25/16 其它错误信息, 查看PatchStatusCode类说明
	                        }
	                    }
	                }).initialize();
initialize()方法内部会强制调用queryNewHotPatch()方法, 所以此处不需要额外再调用queryNewHotPatch()方法



说明： initialize方法

该方法主要做些必要的初始化工作以及如果本地有补丁的话会加载补丁, 所以需要尽可能的早, 推荐在Application的onCreate方法中调用, 由于initialize方法参数越来越多变的原来越臃肿, 所以1.4.0版本修改了调用方式, initialize()方法调用之前你需要先调用如下几个方法, 方法调用说明如下:

	setContext(this): Application上下文context 必选
	
	setAppVersion(appVersion): 应用的版本号 必选
	
	setAppId(appId): 百川上应用的唯一标识, 如何获取请查询获取SDK配置信息 必选
	
	setAesKey(必须16位): 用户自定义aes秘钥, 此时平台无感知这个秘钥, 所以不用担心百川平台会利用你们的补丁做一些非法的事情. 这个参数值必须配合补丁工具的-y参数一起使用, 具体使用参见?Part2 生成patch补丁?的说明, 两者的值需要保持一致, 补丁才能正确被解密进而加载. 可选
	
	setSupportHotpatch(true/false): 目前的版本热修复方案采用类似andfix本地hook方法方法所以热部署有一定的风险(方法正在被运行然后被patch了可能会导致native层的crash). 用户如果有实时生效的需求以及被patch的方法没有被高频调用那么这个参数可以设置为true. 第一个补丁将会即时生效 可选
	
	setEnableDebug(true/false): 是否调试模式, 调试模式下会输出日志以及不进行补丁签名校验. 线下调试此参数可以设置为true, 查看日志过滤TAG:BCHotfix, 同时强制不对补丁进行签名校验, 所有就算补丁未签名或者签名失败也发现可以加载成功. 但是正式发布该参数必须设置为false, 需要对补丁签名校验, 否则就可能存在安全漏洞风险 可选
	
	setPatchLoadStatusStub(new PatchLoadStatusListener()): 设置patch加载状态监听器, 该方法参数需要实现PatchLoadStatusListener接口, 接口说明见1.3.2.2说明 可选 

* initialize(): sdk初始化方法 必选

* PatchLoadStatusListener接口

	* 该接口需要自行实现并传入initialize方法中, 补丁加载状态会回调给该接口, 参数说明如下:

mode: 补丁模式, 0:正常请求模式 1:扫码模式 2:本地补丁模式
code: 补丁加载状态码, 详情查看PatchStatusCode类说明
info: 补丁加载详细说明, 详情查看PatchStatusCode类说明
handlePatchVersion: 当前处理的补丁版本号, 0:无 -1:本地补丁 其它:后台补丁

这里列举几个常见的code码说明, 详情查看SDK中PatchStatusCode类说明

* code: 1 补丁加载成功

* code: 6 服务端没有最新可用的补丁
* code: 11 RSASECRET错误，官网中的密钥是否正确请检查
* code: 12 当前应用已经存在一个旧补丁, 应用重启尝试加载新补丁
* code: 13 补丁加载失败, 导致的原因很多种, 比如UnsatisfiedLinkError等异常, 此时应该严格检查logcat异常日志
* code: 16 APPSECRET错误，官网中的密钥是否正确请检查
* code: 18 一键清除补丁
* code: 403 签名不匹配,可能是APPID APPSECRET填错，请检测

#### queryNewHotPatch方法

该方法主要用于查询服务器是否有新的可用补丁.

* 首先initialize()方法内部会强制调用queryNewHotPatch()方法, 所以initialize()方法调用之后不需要再调用这个方法, 但是你可以在其它你需要的地方调用. 

* 同时SDK内部限制连续两次queryNewHotPatch()方法调用不能短于3s, 否则的话就会报code:19的错误码. 如果查询到可用的话, 首先下载补丁到本地, 然后应用原本没有补丁, 那么第一个补丁会立刻加载
应用已经存在一个补丁, 首先会把之前的补丁文件删除, 然后不立刻加载, 而是等待下次应用重启再加载该补丁
补丁在后台发布之后, 并不会主动下行推送到客户端, 需要手动调用queryNewHotPatch方法查询后台补丁是否可用.

* 只会下载补丁版本号比当前应用存在的补丁版本号高的补丁, 比如当前应用已经下载了版本号为5的补丁, 那么只有后台发布的补丁版本号>5才会重新下载.

* 同时1.4.0版本服务后台上线了“一键清除”补丁的功能, 所以如果后台点击了“一键清除”那么这个方法将会返回code:18的状态码. 此时本地补丁将会被强制清除, 同时不清除本地补丁版本号



#### cleanPatches(boolean force)方法

* 参数force表示是否强制清空本地补丁版本号, 比如当前本地补丁版本号是10, 那么下次再次调用queryNewHotPatch方法时, 如果该参数为false: 不清除本地补丁版本号那么后台最新的补丁1就不会重新下载 

* 当然如果存在比10大的补丁版本仍然是可以下载下来的. 如果该参数为true: 清除本地补丁版本号, 本地补丁版本号将会被设置为0, 所以后台只要有任何发布的补丁都能够下载下来.





 


 
