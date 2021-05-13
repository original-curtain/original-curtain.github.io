---
title: AndroidApp启动流程
date: 2021-05-08 11:34:08
tags: Android
---
首先，桌面的app图标由系统自带的Launcher应用程序管理。在点击桌面图标启动app时，其大致流程如下：
- Launcher启动app并指定启动页面
- AMS记录启动页面的相关信息，并通知Launcher进入pause状态
- Launcher进入pause状态，并通知AMS
- AMS启动新进程
- app启动ActivityThread，并创建application，通知AMS
- AMS发送启动MainActivity的消息
- app启动MainActivity

<!--more-->
Launcher 进程向 AMS 请求创建根 Activity，如果待启动的 Activity 的应用程序未启动，则会通过 Zygote 进程 fork 自身创建应用程序进程，当应用程序进程启动后，AMS 会通过进程间通信启动 Activity。