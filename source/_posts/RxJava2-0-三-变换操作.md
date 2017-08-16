---
title: RxJava2.0(三)变换操作
date: 2017-08-12 15:58:33
tags: [RxJava]
---

了解了线程控制的基本使用，接下来就来看看RxJava厉害的地方--变换操作。

RxJava提供对事件序列进行变换操作。就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。

* map
	
		@Test
	    public void testMap() throws Exception {
	        Observable.create(new ObservableOnSubscribe<String>() {
	            @Override
	            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
	                e.onNext("map");
	            }
	        }).map(new Function<String, Integer>() {
	            @Override
	            public Integer apply(@NonNull String s) throws Exception {
	                return getValue(s);
	            }
	        }).subscribe(new Observer<Integer>() {
	            @Override
	            public void onSubscribe(@NonNull Disposable d) {
	
	            }
	
	            @Override
	            public void onNext(@NonNull Integer s) {
	                System.out.println("testMap:" + s);
	            }
	
	            @Override
	            public void onError(@NonNull Throwable e) {
	
	            }
	
	            @Override
	            public void onComplete() {
	
	            }
	        });
	    }
	    
从上面的例子可以看到，map() 方法将参数中的 String 对象转换成一个 Bitmap 对象后返回，而在经过 map() 方法后，事件的参数类型也由 String 转为了 Bitmap。这种直接变换对象并返回的，是最常见的也最容易理解的变换。
