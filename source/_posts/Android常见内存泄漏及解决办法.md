---
title: Android常见内存泄漏及解决办法
date: 2018-03-13 16:12:15
tags: Android进阶
---

#### Why?

当一个对象已经不需要再使用了，本该被回收时，而有另外一个正在使用的对象持有它的引用从而导致它不能被回收，这导致本该被回收的对象不能被回收而停留在堆内存中，这就产生了内存泄漏。

内存泄漏是造成应用程序OOM的主要原因之一！我们知道Android系统为每个应用程序分配的内存有限，而当一个应用中产生的内存泄漏比较多时，这就难免会导致应用所需要的内存超过这个系统分配的内存限额，这就造成了内存溢出而导致应用Crash。


#### How?

主要几点：

 * Context
 * 内部类（handler等）
 * Cursor
 * Adapter
 * Bitmap

##### 1. 单例造成的内存泄漏

单例的静态特性使得单例的生命周期和应用的生命周期一样长，这就说明了如果一个对象已经不需要使用了，而单例对象还持有该对象的引用，那么这个对象将不能被正常回收，这就导致了内存泄漏。 

例如：

	public class AppManager {
	    private static AppManager instance;
	    private Context context;
	    private AppManager(Context context) {
	        this.context = context;
	    }
	    public static AppManager getInstance(Context context) {
	        if (instance != null) {
	            instance = new AppManager(context);
	        }
	        return instance;
	    }
	}
	
	
   * 传入的是Application的Context：这将没有任何问题，因为单例的生命周期和Application的一样长 

   
   * 传入的是Activity的Context：当这个Context所对应的Activity退出时，由于该Context和Activity的生命周期一样长（Activity间接继承于Context），所以当前Activity退出时它的内存并不会被回收，因为单例对象持有该Activity的引用。 

   
   
 正确的单例：
 
	 public class AppManager {
	    private static AppManager instance;
	    private Context context;
	    private AppManager(Context context) {
	        this.context = context.getApplicationContext();
	    }
	    public static AppManager getInstance(Context context) {
	        if (instance != null) {
	            instance = new AppManager(context);
	        }
	        return instance;
	    }
	}
	
	
	
##### 2.非静态内部类创建静态实例造成的内存泄漏

在实际的项目开发中，有时候我们需要频繁的启动某个页面（Activity），启动的时候总是需要初始化一些资源，为了避免重复创建相同资源，常常会使用静态对象去保存这些值，这种情况下，也很容易照成内存泄漏。我们举个例子：

	public class MainActivity extends AppCompatActivity {
	    private static TestResource mResource = null;
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        if(mManager == null){
	            mManager = new TestResource();
	        }
	        //...
	    }
	    private class TestResource {
	        //...
	    }
	}
	
	
我们创建了一个静态的资源对象mResouce，每次Activity启动都会使用该资源的数据，避免了重复创建。但是这样会造成内存泄漏

* 非静态内部类默认会持有外部类的引用
* 又使用了该非静态内部类创建了一个静态的实例
* 该静态实例的生命周期和应用的一样长，这就导致了该静态实例一直会持有该Activity的引用，导致Activity的内存资源不能正常回收。


解决办法：

* 将内部类testResource改成静态内部类
* 将testResource抽取出来，封装成一个单例


		public class MainActivity extends AppCompatActivity {
		    private static TestResource mResource = null;
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		        if(mManager == null){
		            mManager = new TestResource();
		        }
		        //...
		    }
		    private static class TestResource {
		        //...
		    }
		}
		
		
##### 3. handler导致内存泄漏

举例如下

	public class MainActivity extends AppCompatActivity {
	    private Handler mHandler = new Handler() {
	        @Override
	        public void handleMessage(Message msg) {
	            //...
	        }
	    };
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        loadData();
	    }
	    private void loadData(){
	        //...request
	        Message message = Message.obtain();
	        mHandler.sendMessage(message);
	    }
	}
	
由于mHandler是Handler的非静态匿名内部类的实例，所以它持有外部类Activity的引用，我们知道消息队列是在一个Looper线程中不断轮询处理消息，那么当这个Activity退出时消息队列中还有未处理的消息或者正在处理消息，而消息队列中的Message持有mHandler实例的引用，mHandler又持有Activity的引用，所以导致该Activity的内存资源无法及时回收，引发内存泄漏


解决方案：

 * 在Activity onDestroy的时候，也应该对消息队列中的消息移除。


		 @Override
		public void onDestroy() {
		   //三种均可
		    mHandler.removeMessages(message);
		    mHandler.removeCallbacks(Runnable);
		    mHandler.removeCallbacksAndMessages(null);
		}
		
		
##### 4.资源未关闭导致内存泄漏

BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄漏。