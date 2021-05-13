---
title: Android性能优化
date: 2021-05-13 12:22:26
tags: Android
---
Android性能优化主要从这几个大方面来考量：内存优化，UI渲染性能优化，计算性能优化，电量优化，网络优化，图片优化等
<!--more-->

# 内存优化
主要是针对常见的内存泄露情况的优化：

## 使用单例模式
```
    private static ComonUtil mInstance = null;
    private Context mContext = null;

    public ComonUtil(Context context) {
        mContext = context;
    }

    public static ComonUtil getInstance(Context context) {
        if (mInstance == null) {
            mInstance = new ComonUtil(context);
        }
        return mInstance;
    }
```
其context换成context.getApplicationContext()

## 使用非静态内部类
我们就以线程为例，当Activity调用了createNonStaticInnerClass方法，然后退出当前Activity时，因为线程还在后台执行且当前线程持有Activity引用，只有等到线程执行完毕，Activitiy才能得到释放，导致内存泄漏。
常用的解决方法有很多，第一把线程类声明为静态的类，如果要用到Activity对象，那么就作为参数传入且为WeakReference,第二在Activity的onDestroy时，停止线程的执行。

## 使用异步事件处理机制Handler
如果当handler中处理的是耗时操作，或者当前消息队列中消息很多时，那当Activity退出时，当前message中持有handler的引用，handler又持有Activity的引用，导致Activity不能及时的释放，引起内存泄漏的问题。
解决handler引起的内存泄漏问题常用的两种方式：
1.和上面解决Thread的方式一样，
2.在onDestroy中调用mHandler.removeCallbacksAndMessages(null)

## 使用静态变量
同单例引起的内存泄漏

## 资源未关闭
常见的就是数据库游标没有关闭，对象文件流没有关闭，主要记得关闭就OK了。

## 设置监听
常见的是在观察者模式中出现，我们在退出Acviity时没有取消监听，导致被观察者还持有当前Activity的引用，从而引起内存泄漏。
常见的解决方法就是在onPause中注消监听

## 使用AsyncTask
和上面同样的道理，匿名内部类持有外部类的引用，AsyncTask耗时操作导致Activity不能及时释放，引起内存泄漏。
解决方法同上:
1.声明为静态类，
2.在onPause中取消任务

## 使用Bitmap
我们知道当bitmap对象没有被使用(引用)，gc会回收bitmap的占用内存，当时这边的内存指的是java层的，那么本地内存的释放呢？我们可以通过调用bitmap.recycle()来释放C层上的内存，防止本地内存泄漏