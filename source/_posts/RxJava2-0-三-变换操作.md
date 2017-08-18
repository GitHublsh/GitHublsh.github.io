---
title: RxJava2.0(三)变换操作
date: 2017-08-12 15:58:33
tags: [RxJava]
---

了解了线程控制的基本使用，接下来就来看看RxJava厉害的地方--变换操作。

RxJava提供对事件序列进行变换操作。就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。

#### 一、map


返回一个Observable，它将指定的函数应用于源ObservableSource发出的每个项目，并发出这些函数应用程序的结果。

一对一的变换，如下图（来源：官方文档）

![map](http://ot29getcp.bkt.clouddn.com/images/map.png)
	
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

* 对上游发送的每一个事件应用一个函数, 使得每一个事件都按照指定的函数去变化

* map是一对一的， 可以将上游发来的事件转换为任意的类型, 可以是一个Object, 也可以是一个集合

#### 二、flatmap

	
更加高级的变换。如图（来源：官方文档）
	
![flatmap](http://ot29getcp.bkt.clouddn.com/images/flatMap.png)
	
	
* 一个Observable它发射一个数据序列，这些数据本身也可以发射Observable。RxJava的flatMap()函数提供一种铺平序列的方式，然后合并这些Observables发射的数据，最后将合并后的结果作为最终的Observable。

* flatMap()不能够保证在最终生成的Observable中源Observables确切的发射顺序。


For Example：

	 @Test
    public void testRxFlatMap() throws Exception {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(99);
                e.onNext(66);
                e.onComplete();
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer s) throws Exception {
                if (s>80){
                    return Observable.just("A");
                }
                return Observable.just("B");
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull String s) {
                System.out.println("成绩为："+s);
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
    }
    
    
  从上面的代码很容易看出FlatMap将int变换为String对象，操作简单。一个操作符搞定，这样就方便多了。
  
 
三、Zip

返回一个Observable，它发出指定的组合器函数的结果，该结果应用于依次发送的其他ObservableSource的迭代项的组合。

zip以严格的顺序应用此功能，因此新的ObservableSource发出的第一个项目将是应用于每个源ObservableSources发出的第一个项目的函数的结果; 新的ObservableSource发出的第二个项目将是应用于每个ObservableSource发出的第二个项目的函数的结果; 等等。

来看一个简单的例子，加深理解。

	/**
     * RxJava zip变换
     * @throws Exception
     */
    @Test
    public void testZip() throws Exception {
        Observable<String> observableHello = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("Hello");
            }
        });
        Observable<String> observableWorld = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("World");
            }
        });
        Observable.zip(observableHello, observableWorld, new BiFunction<String, String, String>() {
            @Override
            public String apply(@NonNull String s, @NonNull String s2) throws Exception {
                return s+s2;
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull String s) {
                System.out.println("Final:"+s);
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
    }
    
 从上面的例子就可以看出zip将获取的不同两个String重新组装得到一个新的组装后的String，达到zip类似打包的效果，很好理解。