---
title: RxJava2.0(二)线程控制
date: 2017-08-11 19:27:52
tags: [RxJava]
---

已经知道了基本使用，那就继续进阶更高级的操作-线程切换。

以Android为例，一个Activity的所有动作默认都是在主线程中运行的, 比如：

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.d(TAG, Thread.currentThread().getName());
    }

结果：main.

那么在使用RxJava情况下，在主线程创建Observable发射数据，那么发射源就会在主线程发射数据，在主线程创建Observer接收数据，那么接收源就会在主线程接收数据。
比如：

	@Override                                                                                       
	protected void onCreate(Bundle savedInstanceState) {                                            
	    super.onCreate(savedInstanceState);                                                         
	    setContentView(R.layout.activity_main);                                                     

    Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {   
        @Override                                                                               
        public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {            
            Log.d(TAG, "Observable thread is : " + Thread.currentThread().getName());           
            Log.d(TAG, "emit on main thread");                                                               
            emitter.onNext(1);                                                                  
        }                                                                                       
    });                                                                                         

    Consumer<Integer> consumer = new Consumer<Integer>() {                                      
        @Override                                                                               
        public void accept(Integer integer) throws Exception {                                  
            Log.d(TAG, "Observer thread is :" + Thread.currentThread().getName());              
            Log.d(TAG, "onNext: " + integer);                                                   
        }                                                                                       
    };                                                                                          

    observable.subscribe(consumer);                                                             
	}


即在主线程中创建Observable和Obsever，通过订阅关联后，打印结果显示：
	
	D/TAG: Observable thread is : main
	D/TAG: emit on main thread                    
	D/TAG: Observer thread is :main   
	D/TAG: onNext: 1
	
	
发射源和接收源都在主线程工作。

然而，我们工作的实际情况是，耗时的操作我们会在子线程处理，处理完再到主线程更新UI。

那么，为了达到这样的效果，首先就需要改变上游发送数据的线程，然后下游在主线程接收数据，更新UI。


	@Override                                                                                       
	protected void onCreate(Bundle savedInstanceState) {                                            
	    super.onCreate(savedInstanceState);                                                         
	    setContentView(R.layout.activity_main);                                                     

    Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {   
        @Override                                                                               
        public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {            
            Log.d(TAG, "Observable thread is : " + Thread.currentThread().getName());           
            Log.d(TAG, "emit on new Thread");                                                               
            emitter.onNext(1);                                                                  
        }                                                                                       
    });                                                                                         

    Consumer<Integer> consumer = new Consumer<Integer>() {                                      
        @Override                                                                               
        public void accept(Integer integer) throws Exception {                                  
            Log.d(TAG, "Observer thread is :" + Thread.currentThread().getName());              
            Log.d(TAG, "onNext: " + integer);                                                   
        }                                                                                       
    };                                                                                          

    observable.subscribeOn(Schedulers.newThread())                                              
            .observeOn(AndroidSchedulers.mainThread())                                          
            .subscribe(consumer);                                                               
	}
	
	
结果：

	D/TAG: Observable thread is : RxNewThreadScheduler-2  
	D/TAG: emit on new thread                                         
	D/TAG: Observer thread is :main                       
	D/TAG: onNext: 1
	
从结果就可以看出，上游发射源是在一个新的子线程进行数据的相关处理的。处理后，下游的接收源在主线程接收数据。实现主线程更新UI，子线程处理耗时操作的场景。

仔细观察下，在订阅的时候多了两个操作：

* subscribeOn
* observeOn

下面解释下这两个操作。

* subscribeOn：上游发射源切换发射线程，多次切换的情况仅第一次有效。
* observeOn:下游接收事件线程，可多次指定，没指定一次，就切换一次。



