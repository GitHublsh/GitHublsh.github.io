---
title: RxJava2.0(四)过滤操作
date: 2017-08-22 22:33:27
tags: [RxJava]
---


#### 一、buffer

可以理解为缓存。它定期从Observable收集数据到一个集合，然后把这些数据打包发射，而不是一次发一个。

![buffer](http://ot29getcp.bkt.clouddn.com/images/buffer.png)


#### 二、filter
 
 简单的说，就是按照自定义条件过滤。官方解释：Filters items emitted by an ObservableSource by only emitting those that satisfy a specified predicate.

![filter](http://ot29getcp.bkt.clouddn.com/images/filter.png)

举一个简单的例子：

	@Test
    public void testFilter() throws Exception {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(666);
                e.onNext(6);
                e.onComplete();
            }
        }).filter(new Predicate<Integer>() {
            @Override
            public boolean test(@NonNull Integer integer) throws Exception {
                return integer>100;
            }
        }).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull Integer integer) {
                System.out.println("result:"+integer);
            }

            @Override
            public void onError(@NonNull Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });


从上面的例子，很明显的看出filter按照自己的定义，过滤掉了小于100的数字，然后输出自己想要得到的数字。很容易理解。
