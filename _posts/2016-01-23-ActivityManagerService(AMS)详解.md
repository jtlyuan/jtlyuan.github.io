---
layout: post
title: "ActivityManager和ActivityManagerService(AMS)"
date: 2016-01-26 22:36:00
categories: android
---

* 官方的解析是：作用是与系统运行的Activity进行管理和维护，提供了很多的getXX的方法来获取Activity的Task、Memory扥信息。

* 通过getXX的方法都是通过ActivityManagerNative.getDefault()来实现的。比如
	
		public List<RunningAppProcessInfo> getRunningAppProcesses() {
          try {
              return ActivityManagerNative.getDefault().getRunningAppProcesses();
          } catch (RemoteException e) {
              return null;
          }
        }

* ActivityManagerNativie#getDefault()是什么？直接看代码
    
     	
    	/**
     	* Retrieve the system's default/global activity manager.
     	*/
    	static public IActivityManager getDefault() {
        	return gDefault.get();
    	}
    	
    	private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
        protected IActivityManager create() {
            IBinder b = ServiceManager.getService("activity");
            if (false) {
                Log.v("ActivityManager", "default service binder = " + b);
            }
            IActivityManager am = asInterface(b);
            if (false) {
                Log.v("ActivityManager", "default service = " + am);
            }
            return am;
        }
        };
	
>明显是通过ServiceManager.getService("activity")获取到系统默认的Activity管理类，也就是ActivityManagerService。**所以ActivityManager是通过ActivityManagerService实现对Activity的管理**


* 现在我们分析一下ActivityManagerNative类。定义如下：

```		    
public abstract class ActivityManagerNative extends Binder implements IActivityManager {
	///...
}
```
看下ActivityManagerNative的定义可以看出，ActivityManagerNative是一个Binder，实现了I
ActivityManager接口,和典型的Binder一样，ActiManagerNativive有一个代理内部类ActivityManagerProxy，定义如下

```
class ActivityManagerProxy implements IActivityManager {
	//...
}
```

> 其实这是个经典的Binder调用，ActivityManagerNative和ActivityManagerProxy都实现了IActivityManager接口，ActivityManagerProxy代理类是ActivityManagerNative的内部类也实现了IActivityManager。ActivityManagerService继承ActivityManagerNative，实现IActivityManager的方法。

他们之间的调用关系如下：

![image](https://github.com/jtlyuan/jtlyuan.github.io/blob/master/css/pics/ams-binder%E5%AE%9E%E7%8E%B0.jpg?raw=true)