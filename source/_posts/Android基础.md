---
title: Android基础
date: 2021-04-29 09:18:39
tags: Android
---
待写
<!--more-->
# Android中两种序列化方式的比较Serializable和Parcelable
1. Serializable在序列化的时候会产生大量的临时变量，从而引起频繁的GC，而相比之下Parcelable的性能更高(毕竟是Android自带的)，所以当在使用内存时(如：序列化对象在网络中传递对象或序列化在进程间传递对象)，更推荐使用Parcelable接口。 
2. 但Parcelable有个明显的缺点：不能能使用在要将数据存储在磁盘上的情况(如：永久性保存对象，保存对象的字节序列到本地文件中)，因为Parcel本质上为了更好的实现对象在IPC间传递，并不是一个通用的序列化机制，当改变任何Parcel中数据的底层实现都可能导致之前的数据不可读取，所以此时还是建议使用Serializable 。

# Intent和Bundle的区别
Intent旨在数据传递，bundle旨在存取数据，

intent内部还是用bundle来实现数据传递的，只是封装了一层而已。

在使用的时候如果需要传递的数据比较多，还是用Bundle来存储数据比较好。毕竟人家是专门做这个的。还有一个好处就是，如果您在ABC三个页面中传值且顺序必须是ABC，直接传递Bundle的数据就好了。而不用在 B 将数据从Intent拿出来,然后封装到新的Intent，传递到C，多此一举。