---
title: Android四大组件
date: 2021-04-29 09:01:59
tags: Android
---
待写
<!--more-->
# Service
## 服务中onStartCommand方法返回值的作用
服务中 int onStartCommand(Intent intent, int flag, int startID)有4种返回值：当Service被异常kill掉后，如有异常抛出。
- Service.START_STICKY：系统会自动偿试重新启动服务，为intent传值null
- START_REDELIVER_INTENT：系统会自动偿试重新启动服务，并为intent传入Service被kill之前的intent的值。
- START_NOT_STICKY：系统不会自动重启该服务。
- START_STICKY_COMPATIBILITY：START_STICKY的兼容版本，但不保证服务被kill后一定能重启

## bindService是一个异步过程
Android中bindService是一个异步的过程，什么意思呢？使用bindService无非是想获得一个Binder服务的Proxy，但这个代理获取到的时机并非由bindService发起端控制，而是由Service端来控制，也就是说bindService之后，APP端并不会立刻获得Proxy，而是要等待Service通知APP端，具体流程可简化如下：
- APP端先通过bindService去AMS登记，说明自己需要绑定这样一个服务，并留下派送地址
- APP回来，继续做其他事情，可以看做是非阻塞的
- AMS通知Service端启动这个服务
- Service启动，并通知AMS启动完毕
- AMS跟住之前APP端留下的地址通知APP端，并将Proxy代理传递给APP端

## 是否能在Service进行耗时操作？如果非要可以怎么做，如何避免service线程卡顿？service里面可以弹土司吗？
是否能在Service进行耗时操作？
- 默认情况,如果没有显示的指定service所运行的进程,Service和Activity是运行在当前app所在进程的mainThread(UI主线程)里面。
- service里面不能执行耗时的操作(网络请求,拷贝数据库,大文件)，在Service里执行耗时操作，有可能出现主线程被阻塞（ANR）的情况。


如果非要可以怎么做，如何避免service线程卡顿？
- 需要在子线程中执行 new Thread(){}.start();


service里面可以弹土司吗？
可以的。弹吐司有个条件就是得有一个 Context 上下文，而 Service 本身就是 Context 的子类，因此在 Service 里面弹吐司是完全可以的。比如我们在 Service 中完成下载任务后可以弹一个吐司通知用户。

## IntentService
IntentService 是 Service 的子类，比普通的 Service 增加了额外的功能。
先看 Service 本身存在两个问题：
- Service 不会专门启动一条单独的进程，Service 与它所在应用位于同一个进程中；
- Service 也不是专门一条新线程，因此不应该在 Service 中直接处理耗时的任务；

IntentService 特征
- 会创建独立的 worker 线程来处理所有的 Intent 请求；
- 会创建独立的 worker 线程来处理 onHandleIntent()方法实现的代码，无需处理多线程问题；
- 所有请求处理完成后，IntentService 会自动停止，无需调用 stopSelf()方法停止 Service；
- 为Service 的 onBind()提供默认实现，返回 null；
- 为Service 的 onStartCommand 提供默认实现，将请求 Intent 添加到队列中

## Activity如何与Service通信
- 添加一个继承Binder的内部类，并添加相应的逻辑方法。重写Service的onBind方法，返回我们刚刚定义的那个内部类实例。Activity中创建一个ServiceConnection的匿名内部类，并且重写里面的onServiceConnected方法和onServiceDisconnected方法，这两个方法分别会在活动与服务成功绑定以及解除绑定的时候调用，在onServiceConnected方法中，我们可以得到一个刚才那个service的binder对象，通过对这个binder对象进行向下转型，得到我们那个自定义的Binder实例，有了这个实例，做可以调用这个实例里面的具体方法进行需要的操作了
- 通过BroadCast(广播)的形式，当我们的进度发生变化的时候我们发送一条广播，然后在Activity的注册广播接收器，接收到广播之后更新视图