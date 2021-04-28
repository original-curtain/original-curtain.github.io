---
title: Java数据结构
date: 2021-04-28 19:20:09
tags: Java
---
JDK提供了一组主要的数据结构实现，如List、Map、Set、Queue 等常用数据结构。这些数据都继承自java.util.Collection接口，并位于java.util包内
<!--more-->
# List
## ArrayList
- 基于动态数组集合，可以动态的增加、删除元素，动态扩容等，默认初始容量10，超出上限会扩容至： int newCapacity = oldCapacity + (oldCapacity >> 1); 也就是原来容量的1.5倍
- 优点：按下标索引速度快(O(1))。缺点：插入删除会慢(O(n))。

###  在arrayList中System.arraycopy()和Arrays.copyOf()方法区别联系
- copyOf()内部调用了System.arraycopy()方法
- 区别：
    1. arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置
    2. copyOf()是系统自动在内部新建一个数组，并返回该数组。

## LinkedList
- 基于链表的集合，是一个双向链表，没有初始化大小，也没有扩容的机制，会一直在前面或者后面新增Node。
- 优点：便于存取，只要改变头尾节点指向 (O(1))。缺点：索引慢，极端情况会出现从头结点遍历到最后一个节点的情况 (O(n))

## Vector
- 也是基于数组的一个集合，对一些操作数据的方法加了synchronized，可以看做是ArrayList一个同步、线程安全的版本
- 优点：线程安全的。缺点：同步必然会导致效率慢。

## CopyOnWriteArrayList
- CopyOnWriteArrayList是ArrayList的一个线程安全的变体，其中所有可变操作（add、set等等）都是通过对底层数组进行一次新的复制来实现的。相比较于ArrayList它的写操作要慢一些，因为它需要实例的快照
- CopyOnWriteArrayList中写操作需要大面积复制数组，所以性能肯定很差，但是读操作因为操作的对象和写操作不是同一个对象，读之间也不需要加锁，读和写之间的同步处理只是在写完后通过一个简单的"="将引用指向新的数组对象上来，这个几乎不需要时间，这样读操作就很快很安全，适合在多线程里使用，绝对不会发生ConcurrentModificationException ，因此CopyOnWriteArrayList适合使用在读操作远远大于写操作的场景里，比如缓存

## Arrays.asList方法后的List可以扩容吗
不能，asList返回的List为只读的。其原因为：asList方法返回的ArrayList是Arrays的一个内部类，并且没有实现add，remove等操作

# Map
## HashMap
- HashMap 底层是数组+链表(jdk1.8是数组+链表/红黑树)，HashMap可能也是应用最多的数据结构了。
- 允许存在一个为null的key和任意个为null的value
- 采用链表散列的数据结构，即数组和链表的结合；初始容量为16，填充因子默认为0.75，扩容时是当前容量翻倍，即2capacity

### HashMap存储两个对象的hashcode相同会发生什么
因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中。

### 如果两个键的hashcode相同，你如何获取值对象
- 当调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。当然如果有两个值对象储存在同一个bucket，将会遍历链表直到找到值对象。
- 在没有值对象去比较，如何确定确定找到值对象的？因为HashMap在链表中存储的是键值对，找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点，最终找到要找的值对象。

### HashMap为什么不直接使用hashCode()处理后的哈希值直接作为table的下标
hashCode()方法返回的是int整数类型，其范围为-(2^31)~(2^31-1)，约有40亿个映射空间，而HashMap的容量范围是在16（初始化默认值）~2 ^ 30，HashMap通常情况下是取不到最大值的，并且设备上也难以提供这么多的存储空间，从而导致通过hashCode()计算出的哈希值可能不在数组大小范围内，进而无法匹配存储位置；

### HashMap是使用了哪些方法来有效解决哈希冲突的
1. 使用链地址法（使用散列表）来链接拥有相同hash值的数据；
2. 使用2次扰动函数（hash函数）来降低哈希冲突的概率，使得数据分布更平均；
3. 引入红黑树进一步降低遍历的时间复杂度，使得遍历更快

### 为什么HashMap中String、Integer这样的包装类适合作为K？为啥不用其他作key值？
- 都是final类型，即不可变性，保证key的不可更改性，不会存在获取hash值不同的情况
- 内部已重写了equals()、hashCode()等方法，遵守了HashMap内部的规范（不清楚可以去上面看看putValue的过程），不容易出现Hash值计算错误的情况；

### 想要让自己的Object作为K应该怎么办呢
重写hashCode()和equals()方法

## SparseArray
- 位于android.util，Android 中的数据结构，针对移动端做了优化，在数据量比较少的情况下，性能会好过 HashMap，类似于 HashMap，key:int ,value:object 。
    1. key  和 value  采用数组进行存储。存储 key 的数组是 int 类型，不需要进行装箱操作。提供了速度。
    2. 采用二分查找法，在插入进行了排序，所以两个数组是按照从小到大进行排序的。
    3. 在查找的时候，进行二分查找，数据量少的情况下，速度比较快。

## LinkedHashMap
HashMap是无序的，而LinkedHashMap是有序的HashMap，默认为插入顺序，还可以是访问顺序，基本原理是其内部通过Entry维护了一个双向链表，负责维护Map的迭代顺序

## HashTable
- 其实就是HashMap的一个线程安全版本，像Vector和ArrayList的关系一样，对内部的方法都加了synchronized关键字修饰。
- 不允许使用null作为key和value
- 初始容量为11，填充因子默认为0.75，扩容时是容量翻倍+1，即2capacity+1
- 缺点：因为采用synchronized保证同步，每次都会锁住整个map，所以高并发线程在争夺同一把锁的时候性能急剧下降

## TreeMap
- 平常开发中用的不多，是一个红黑树版本的map，实现了NavigableMap<K,V>并且NavigableMap又继承了SortedMap<K,V>类，SortedMap接口有一个Comparator<? super K> comparator();比较器，所以TreeMap是支持比较排序的。

## ConcurrentHashMap
- 底层也是HashMap，同时保证了线程安全，与HashTable不同的ConcurrentHashMap采用分段锁思想，抛弃了使用synchronized修饰操作方法的同步方式。
- 1.7与1.8的区别：
    1. 因为底层是HashMap，so 1.8之后也变成了数组+链表/红黑树。
    2. 1.8之后放弃了分段锁，采用了synchronized+CAS来保证并发
    - 1.8为何放弃分段锁： 
        1. 我认为主要是1.8对synchronized进行了优化(偏向锁、轻量级锁、自旋锁、自适宜自旋)
        2. 加入多个分段锁浪费内存空间。
        3. 生产环境中， map在放入时竞争同一个锁的概率非常小，分段锁反而会造成更新等操作的长时间等待。

# Set
## HashSet
- HashSet 基于HashMap实现，利用Map的key不能重复来实现Set的元素唯一性

## LinkedHashSet
- LinkedHashSet 基于LinkedHashMap实现，继承自HashSet，可以看出是一个有序的Set（可以像LinkedHashMap自定义排序）

## TreeSet
是SortedSet接口的实现类，根据元素实际值的大小进行排序；采用红黑树的数据结构来存储集合元素；支持两种排序方法：自然排序（默认情况）和定制排序。前者通过实现Comparable接口中的compareTo()比较两个元素之间大小关系，然后按升序排列；后者通过实现Comparator接口中的compare()比较两个元素之间大小关系，实现定制排列

# Stack

# Queue

# Collections.sort和Arrays.sort的区别
- Arrays.sort()是一个经过调优的快速排序
- Collections.sort()是一个经过修改的合并排序算法
- Collections.sort专门给List排序，而Arrays.sort专门给数组进行排序
- Collections.sort排序底层调用的是Arrays.sort方法

# Java集合的快速失败机制 “fail-fast”
- 例如：假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 ConcurrentModificationException 异常，从而产生fail-fast机制。
- 原因：
    - 迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历
- 解决办法：
    1. 在遍历过程中，所有涉及到改变modCount值得地方全部加上synchronized。
    2. 使用CopyOnWriteArrayList来替换ArrayList
