---
title: RxJava2.0(一)
date: 2017-08-15 19:16:59
tags: [RxJava]
---

#### RxJava2.0(一)

--

一、基本概念
	
官方介绍：
	RxJava是Reactive Extensions的Java VM实现：用于通过使用observable序列来组合异步和基于事件的程序的库。
	
简单来说，就是异步，观察者模式。

* 首先需要了解几个概念
	* Observable 被观察者，即数据发射源
	* Observer 观察者，即数据接收源
	* subscribe 订阅，即将数据发射源和接收源相关联

二、基本使用

了解了基本概念，开始基本使用。

写一个测试用例：
	
	/**
     * RxJava基本使用
     */
    @Test
    public void testRxJava() throws Exception {
        Observable<String> observable = new Observable<String>() {
            @Override
            protected void subscribeActual(Observer<? super String> observer) {
                observer.onNext("1");
            }
        };
        Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull String s) {
                System.out.println("Observer:"+s);

            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        };
        observable.subscribe(observer);
    }
    
    
  从上面的例子可以看到很简单的，发射源Observable发射数据，接收源Observer接收数据，通过subscribe将发射源和接收源相关联。订阅后才会开始发射数据。
  
  简化上述操作，就是RxJava比较好的链式操作。
  	
  	/**
     * RxJava链式基本操作
     *
     * @throws Exception
     */
    @Test
    public void testObservable1() throws Exception {
        Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("hello");
                e.onComplete();
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull String s) {
                System.out.println("Observer Receive:" + s);
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
    }


三、简单介绍用法

* 发射源，通过ObservableEmitter发射数据。Emitter的意思是发射器，这个就是用来发出事件的，它可以发出三种类型的事件，通过调用emitter的onNext(T value)、onComplete()和onError(Throwable error)就可以分别发出next事件、complete事件和error事件。
* 上游可以发射无数个onNext事件，当发射onComplete()事件后，下游不再接收上游发送的数据，但是上游还是可以继续发送数据的。
* onComplete和onError必须唯一并且互斥, 即不能发多个onComplete, 也不能发多个onError, 也不能先发一个onComplete, 然后再发一个onError。这一点很重要。



