---
title: Java死锁浅析
date: 2018-11-14 15:38:44
tags: [Java]
---

## 一、死锁的基本概念
死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。”那么我们换一个更加规范的定义：“集合中的每一个进程都在等待只能由本集合中的其他进程才能引发的事件，那么该组进程是死锁的。


产生死锁必须同时满足以下四个条件，只要其中任一条件不成立，死锁就不会发生。

* 互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
* 不剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放)。
* 请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
* 循环等待条件：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被 链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有


```
*public class*SyncABDemo {
    *public static void*main(String args[]){

    Object a = *new*Object();
    Object b = *new*Object();
    Thread threadA = *new*Thread(*new*Runnable() {
        @Override
        *public void*run() {
            *synchronized*(a) {
                *try*{
                    System.*out*.println(*"ThreadA--a is Locked"*);
                    Thread./sleep/(1000);
                    *synchronized*(b) {
                        System.*out*.println(*"ThreadA--b is Locked"*);
                    }
                } *catch*(InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    });
    Thread threadB = *new*Thread(*new*Runnable() {
        @Override
        *public void*run() {
            *synchronized*(b) {
                *try*{
                    System.*out*.println(*"ThreadB--a is Locked"*);
                    Thread./sleep/(1000);
                    *synchronized*(a) {
                        System.*out*.println(*"ThreadB--b is Locked"*);
                    }
                } *catch*(InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    });
    threadA.start();
    threadB.start();
    }
}

```


输出结果为：

```
ThreadA--a is Locked
ThreadB--a is Locked

```

发现程序执行停滞了。

这个时候程序发生了死锁。

## 二、检测死锁
JDK自带检测工具
可以在IDEA命令行输入JConsole打开JConsole连接本地进程就可以进行检测死锁的操作了。


## 三、死锁预防

### 1.以确定的顺序获得锁

如果一定要获取多个锁，那么在设计的时候需要考虑多线程情况下获取锁的情况

例如上述代码示例：

* 线程A -> 获取锁a -> 获取锁b -> 永久等待

* 线程B -> 获取锁b -> 获取锁a -> 永久等待

那么如果把获取锁的时序更改为下面的情况：

* 线程A -> 获取锁a -> 获取锁b -> 结束

* 线程B -> 获取锁a -> 获取锁b -> 结束

那么死锁就不会发生了

### 2.超时放弃

当使用synchronized关键词提供的内置锁时，只要线程没有获得锁，那么就会永远等待下去，然而Lock接口提供了

`boolean tryLock(long time, TimeUnit unit) throws InterruptedException`

方法，该方法可以按照固定时长等待锁，因此线程可以在获取锁超时以后，主动释放之前已经获得的所有的锁。通过这种方式，也可以很有效地避免死锁。

## 四、死锁举例

### 线程池死锁

```
final ExecutorService executorService = 
        Executors.newSingleThreadExecutor();
Future<Long> f1 = executorService.submit(new Callable<Long>() {

    public Long call() throws Exception {
        System.out.println("start f1");
        Thread.sleep(1000);//延时
        Future<Long> f2 = 
           executorService.submit(new Callable<Long>() {

            public Long call() throws Exception {
                System.out.println("start f2");
                return -1L;
            }
        });
        System.out.println("result" + f2.get());
        System.out.println("end f1");
        return -1L;
    }
});
```
