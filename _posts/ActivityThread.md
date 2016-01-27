---
layout: post
title: "ActivityThread解析"
date:2016-1-27 21:08
categories: android
---
#Activity底层工作原理

##ActivityThread
**这是一个管理主线程执行的java普通类。创建Android的UI线程，并处理消息循环的事件。**

这个类提供了Android应用程序入口ActivityThread#main()方法,如下：

	public static void main(String[] args) {
		...
		//创建主线程Looper对象
        Looper.prepareMainLooper();
		//创建ActivityThread对象
        ActivityThread thread = new ActivityThread();
        
        thread.attach(false);//

        if (sMainThreadHandler == null) {
        	//创建主线程的Handler
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }
		//启动主线程looper
	    Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
    
    
从代码可以看出main方法的作用是启动一个UI线程和创建ActivityThread#attach()方法。那么ActivityThread#attach()方法是做什么的呢？直接看代码，如下：
	
	private void attach(boolean system) {
        sCurrentActivityThread = this;
        if (!system) {
        	//ActivityManagerNative.getDefault(）实质是获取一个ActivityManagerService（AMS）
            final IActivityManager mgr = ActivityManagerNative.getDefault();
            try {
            	//mAppThread是ApplicationThread,把AMS和ApplicationThread关联起来
                mgr.attachApplication(mAppThread);
            } catch (RemoteException ex) {
                // Ignore
            }
            
        } else {
             try {
                mInstrumentation = new Instrumentation();
                ContextImpl context = ContextImpl.createAppContext(
                        this, getSystemContext().mPackageInfo);
                mInitialApplication = context.mPackageInfo.makeApplication(true, null);
                mInitialApplication.onCreate();
            } catch (Exception e) {
                throw new RuntimeException(
                        "Unable to instantiate Application():" + e.toString(), e);
            }
        }
    }
    

>1、ApplicationThread 继承了ApplicationManagerNative类，ApplicationManagerNative是一个实现IApplicationThread的abstract类，实现了一堆的IAppscheduleXX()调度方法，实在实现的地方在ApplicationThread类。很明显这是一个Binder调用。

2、ActivityManagerNative.getDefault(）实质是获取一个ActivityManagerService（AMS）,其实这是AMS对UI进行的调用是通过Binder方式进行IPC调用到ApplicationThread，从而实现对UI线程的操作。

