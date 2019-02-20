---
title: 主线程的Loop.loop()一直无限循环为什么不会造成ANR?
date: 2018-05-30 11:27:35
tags: [Android]
---

#### 一、基本概念

##### 什么是ANR?

简单来说就是：

1. 当前事件没有机会得到处理

2. 当前事件正在处理但是没有及时完成

##### 源码分析

ActivityThread的main方法主要就是做消息循环，一旦退出消息循环，那么你的应用也就退出了。

```
public static void main(String[] args) {
		        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
		        SamplingProfilerIntegration.start();
		
	        // CloseGuard defaults to true and can be quite spammy.  We
	        // disable it here, but selectively enable it later (via
	        // StrictMode) on debug builds, but using DropBox, not logs.
	        CloseGuard.setEnabled(false);
	
	        Environment.initForCurrentUser();
	
	        // Set the reporter for event logging in libcore
	        EventLogger.setReporter(new EventLoggingReporter());
	
	        AndroidKeyStoreProvider.install();
	
	        // Make sure TrustedCertificateStore looks in the right place for CA certificates
	        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
	        TrustedCertificateStore.setDefaultUserDirectory(configDir);
	
	        Process.setArgV0("<pre-initialized>");
	
	        Looper.prepareMainLooper();
	
	        ActivityThread thread = new ActivityThread();
	        thread.attach(false);
	
	        if (sMainThreadHandler == null) {
	            sMainThreadHandler = thread.getHandler();
	        }
	
	        if (false) {
	            Looper.myLooper().setMessageLogging(new
	                    LogPrinter(Log.DEBUG, "ActivityThread"));
	        }
	
	        // End of event ActivityThreadMain.
	        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
	        Looper.loop();
	
	        throw new RuntimeException("Main thread loop unexpectedly exited");
	    }
```

进入`Loop.loop()`方法

```
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

//死循环，取出消息队列中的消息，可能会阻塞
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }

```

得出结论：因为Android 的是由事件驱动的，looper.loop() 不断地接收事件、处理事件，每一个点击触摸或者说Activity的生命周期都是运行在 Looper.loop() 的控制之下，如果它停止了，应用也就停止了。只能是某一个消息或者说对消息的处理阻塞了 Looper.loop()，而不是 Looper.loop() 阻塞它。
也就说我们的代码其实就是在这个循环里面去执行的，当然不会阻塞了。



		    
		    
