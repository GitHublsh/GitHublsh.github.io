---
title: 主线程的Loop.loop()一直无限循环为什么不会造成ANR?
date: 2018-05-30 11:27:35
tags: [Android]
---

#### 一、基本概念


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
	    
	    
