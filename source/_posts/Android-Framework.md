---
title: Android Framework
date: 2021-04-25 19:49:59
tags: Android
---
Android系统的构成，从上到下依次是：
- Application应用层：应用层，即我们所安装的所有应用
- Framework框架层：应用框架层，全部是用Java语言编写的，供开发人员调用
- Library系统库层
    - 用C语言编写的完成Android核心功能的一些类库，如：OpenGL|ES（图形图像引擎简化版）、WebKit（浏览器内核）、SQLite（轻量级数据库）、Surface Manager（界面管理器）、Media Framework（多媒体框架）、FreeType（字体类库）、SGL（另一个图形图像引擎）、SSL（基于TCP的安全协议）、libc（零散的类库）。
    - 其中还有AdnroidRuntime
        - Core Libraries：核心类库。
        - Dalvik Virtual Machine：Android底层是Linux系统，使用C、C++语言编写的，所以Android程序（Java语言编写）要在Linux上运行就需要虚拟机，也就是DVM
- Linux内核层
    - Linux核心，Android系统是基于Linux系统修改过来的，Android底层都是Linux的东西，大多都是操作硬件的一些驱动，如Display Driver、Audio Drivers等等。
<!--more-->
# Framework功能
- 用Java语言编写一些规范化的模块封装成框架，供APP层开发者调用开发出具有特殊业务的手机应用。
- 用Java Native Interface调用core lib层的本地方法，JNI的库是在Dalvik虚拟机启动时加载进去的，Dalvik会直接去寻址这个JNI方法，然后去调用。

提供的服务
- Activity Manager ： 用来管理应用程序生命周期并提供常用的导航回退功能
- Window Manager：提供一些我们访问手机屏幕的方法。屏幕的透明度、亮度、背景
- Content Providers：使得应用程序可以访问另一个应用程序的数据（如联系人数据库)， 或者共享它们自己的数据
- View System：可以用来构建应用程序， 它包括列表（Lists)，网格（Grids)，文本框（Textboxes)，按钮（Buttons)， 甚至可嵌入的web浏览器。
- Notification Manager：使得应用程序可以在状态栏中显示自定义的提示信息。
- Package Manager：提供对系统的安装包的访问。包括安装、卸载应用，查询permission相关信息，查询Application相关信息等
- Telephony Manager ：主要提供了一系列用于访问与手机通讯相关的状态和信息的方法，查询电信网络状态信息，sim卡的信息等。
- Resource Manager：提供非代码资源的访问，如本地字符串，图形，和布局文件（Layout files )
- Location Manager：提供设备的地址位置的获取方式。很显然，GPS导航肯定能用到位置服务。
- XMPP：可扩展通讯和表示协议。前身为Jabber，提供即时通信服务。例如推送功能,Google Talk。

三大核心功能：
- View.java:View工作原理，实现包括绘制view、处理触摸、按键事件等。
- ActivityManagerService.java:Ams 管理所有应用程序的Activity等。
- WindowManagerService.java:Wms 为所有应用程序分配窗口，并管理这些窗口。