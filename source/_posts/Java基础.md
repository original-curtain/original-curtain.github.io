---
title: Java基础
date: 2021-04-28 18:36:02
tags: Java
---
# ==和equal()的区别
对于==：
- 基本类型：比较值是否相同
- 引用类型：比较引用是否相同
<!--more-->

对于equal():
```
public boolean equals(Object obj) {
		return (this == obj);
}
```
其本质是==，不过String，Integer等重写了equal()方法，使其变成值比较
```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

# String,StringBuilder,StringBuff区别
## String
- String是不可变字符串，底层是final修饰的字符数组
- String 对象赋值之后就会在字符串常量池中缓存
```
String s1="cat";
/*
此时首先会去字符串常量池中查找看有没有“Cat”字符串，如果有则返回它的地址给s1,如果没有则在常量池中创建“Cat”字符串，并将地址返回给s1
*/
String s3 = new String(“Cat”);
/*
当我们用这种方式创建字符串对象的时候，首先会去字符串常量池中查找看有没有“Cat”字符串,如果没有则在常量池中创建“Cat”字符串，然后在堆内存中创建“Cat”字符串，并将堆内存中的地址返回给s3.
*/
```
## StringBuilder
StringBuilder是非线程安全的,适用于单线程下在字符缓冲区进行大量操作的情况

## StringBuff
StringBuffer是线程安全的,适用于多线程下在字符缓冲区进行大量操作的情况

# 对象克隆
## 方式
- 实现Cloneable接口并重写Object类中的clone()方法；
- 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆

# 深克隆和浅克隆
- 浅克隆：创建一个新对象，新对象的属性和原来对象完全相同，对于非基本类型属性，仍指向原有属性所指向的对象的内存地址。
- 深克隆：创建一个新对象，属性中引用的其他对象也会被克隆，不再指向原有对象地址

# 装箱和拆箱
```
int a = 10;
//装箱操作
Integer integer1 = Integer.valueOf(a);

//拆箱操作
Integer integer2 = new Integer(5);
int i2 = integer2.intValue();
```
## new Integer(123) 与 Integer.valueOf(123)的区别
- new Integer(123) 每次都会新建一个对象；技术博客大总结
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用
```
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

# 通配符

# 泛型擦除

# 线程安全的基本类型
Java自带的线程安全的基本类型包括： AtomicInteger, AtomicLong, AtomicBoolean, AtomicIntegerArray,AtomicLongArray等
```
private static AtomicInteger atomicInteger = new AtomicInteger(1);
static Integer count1 = Integer.valueOf(0);
private void startThread1() {
    for (int i = 0;i < 200; i++){
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int k = 0; k < 50; k++){
                    // getAndIncrement: 先获得值，再自增1，返回值为自增前的值
                    count1 = atomicInteger.getAndIncrement();
                }
            }
        }).start();
    }
    // 休眠10秒，以确保线程都已启动
    try {
        Thread.sleep(1000*10);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }finally {
        Log.e("打印日志----",count1+"");
    }
}

//期望输出10000，最后输出的是10000
//注意：打印日志----: 10000

//AtomicInteger使用了volatile关键字进行修饰，使得该类可以满足线程安全。
private volatile int value;
public AtomicInteger(int initialValue) {
    value = initialValue;
}
```

# transient关键字
- 对于不想进行序列化的变量，使用transient关键字修饰。
- transient关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被transient修饰的变量值不会被持久化和恢复。transient只能修饰变量，不能修饰类和方法