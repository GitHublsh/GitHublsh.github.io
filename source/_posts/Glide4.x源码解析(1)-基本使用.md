---
title: Glide4.x源码解析(1)-基本使用
date: 2017-12-11 16:41:28
tags: [Glide]
---

### 一、关于Glide

* 官方定义：

	* Glide是一个快速高效的Android图片加载库，注重于平滑的滚动。Glide提供了易用的API，高性能、可扩展的图片解码管道（decode pipeline），以及自动的资源池技术。

   * Glide 支持拉取，解码和展示视频快照，图片，和GIF动画。Glide的Api是如此的灵活，开发者甚至可以插入和替换成自己喜爱的任何网络栈。默认情况下，Glide使用的是一个定制化的基于HttpUrlConnection的栈，但同时也提供了与Google Volley和Square OkHttp快速集成的工具库。

	* 虽然Glide 的主要目标是让任何形式的图片列表的滚动尽可能地变得更快、更平滑，但实际上，Glide几乎能满足你对远程图片的拉取/缩放/显示的一切需求。

	
* 性能

	两个关键点：
	* 图片的解码速度
	
	* 解码图片带来的资源压力也就是内存消耗

	Glide使用了多个步骤来确保在Android上加载图片尽可能的快速和平滑：
	
	* 自动、智能地下采样(downsampling)和缓存(caching)，以最小化存储开销和解码次数；
	* 积极的资源重用，例如字节数组和Bitmap，以最小化昂贵的垃圾回收和堆碎片影响；
	* 深度的生命周期集成，以确保仅优先处理活跃的Fragment和Activity的请求，并有利于应用在必要时释放资源以避免在后台时被杀掉。
	

### 二、使用配置

#### 1.依赖

##### Gradle
使用 Gradle，可从 Maven Central 或 JCenter 中添加对 Glide 的依赖。还需要添加 Android 支持库的依赖

	repositories {
	  mavenCentral()
	  maven { url 'https://maven.google.com' }
	}
	
	dependencies {
	    compile 'com.github.bumptech.glide:glide:4.4.0'
	    annotationProcessor 'com.github.bumptech.glide:compiler:4.4.0'
	}

##### Maven

使用 Maven，同样可以添加对 Glide 的依赖。同样需要添加Android支持库的依赖

	<dependency>
	  <groupId>com.github.bumptech.glide</groupId>
	  <artifactId>glide</artifactId>
	  <version>4.4.0</version>
	  <type>aar</type>
	</dependency>
	<dependency>
	  <groupId>com.google.android</groupId>
	  <artifactId>support-v4</artifactId>
	  <version>r7</version>
	</dependency>
	<dependency>
	  <groupId>com.github.bumptech.glide</groupId>
	  <artifactId>compiler</artifactId>
	  <version>4.4.0</version>
	  <optional>true</optional>
	</dependency>

#### 2.相关设置

##### 权限

* 网络

当需要的图片是需要网络的情况，需在AndroidManifest添加网络权限。

	<uses-permission android:name="android.permission.INTERNET"/>

* 连接状态

如果你正在从 URL 加载图片，Glide 可以自动帮助你处理片状网络连接：它可以监听用户的连接状态并在用户重新连接到网络时重启之前失败的请求。如果 Glide 检测到你的应用拥有 ACCESS_NETWORK_STATUS 权限，Glide 将自动监听连接状态而不需要额外的改动。

	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	
* 本地存储

要从本地文件夹或 DCIM 或图库中加载图片，你将需要添加 READ_EXTERNAL_STORAGE 权限。

	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
	
### 三、基本使用

Glide的基本使用很简单：
	
	Glide.with(fragment)
	    .load(myUrl)
	    .into(imageView);
	    
一行就可以搞定从网络加载图片，丝柔顺滑。

具体实现：

	public class MainActivity extends AppCompatActivity {
	
	    Button mBtn;
	    ImageView imageView;
	    String myUrl = "http://ot29getcp.bkt.clouddn.com/map.png";
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        mBtn = (Button)findViewById(R.id.downloadimage);
	        imageView = (ImageView) findViewById(R.id.image);
	        mBtn.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                Glide.with(getApplicationContext())
	                        .load(myUrl)
	                        .into(imageView);
	            }
	        });
	
	    }
	}
	
核心代码：

就这一行。

	Glide.with(getApplicationContext())
		                        .load(myUrl)
		                        .into(imageView);



