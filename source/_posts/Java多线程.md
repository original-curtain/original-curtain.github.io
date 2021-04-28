---
title: Java多线程
date: 2021-04-28 20:05:27
tags: Java
---
# 线程池

# 线程方法
## 线程中start和run方法
- 调用start()方法时你将创建新的线程，并且执行在run()方法里的代码
- 调用run()方法，它不会创建新的线程也不会执行调用线程的代码。

## wait和sleep方法
最大的不同是在等待时wait会释放锁，而sleep一直持有锁。Wait通常被用于线程间交互，sleep通常被用于暂停执行。

## Thread.sleep(0)的作用
由于Java采用抢占式的线程调度算法，因此可能会出现某条线程常常获取到CPU控制权的情况，为了让某些优先级比较低的线程也能获取到CPU控制权，可以使用Thread.sleep(0)手动触发一次操作系统分配时间片的操作，这也是平衡CPU控制权的一种操作。

## yield()方法
yield()方法和sleep()方法类似，也不会释放“锁标志”，区别在于，它没有参数，即yield()方法只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行，另外yield()方法只能使同优先级或者高优先级的线程得到执行机会，这也和sleep()方法不同。

## join()方法
Thread的join()的含义是等待该线程终止，即将挂起调用线程的执行，直到被调用的对象完成它的执行。比如存在两个线程t1和t2，下述代码表示先启动t1，直到t1的任务结束，才轮到t2启动。
```
t1.start();
t1.join(); 
t2.start();
```

# 死锁
线程A和线程B相互等待对方持有的锁导致程序无限死循环下去
```
public class DeadLock {
    public static String obj1 = "obj1";
    public static String obj2 = "obj2";
    public static void main(String[] args){
        Thread a = new Thread(new Lock1());
        Thread b = new Thread(new Lock2());
        a.start();
        b.start();
    }    
}
class Lock1 implements Runnable{
    @Override
    public void run(){
        try{
            System.out.println("Lock1 running");
            while(true){
                synchronized(DeadLock.obj1){
                    System.out.println("Lock1 lock obj1");
                    Thread.sleep(3000);//获取obj1后先等一会儿，让Lock2有足够的时间锁住obj2
                    synchronized(DeadLock.obj2){
                        System.out.println("Lock1 lock obj2");
                    }
                }
            }
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}
class Lock2 implements Runnable{
    @Override
    public void run(){
        try{
            System.out.println("Lock2 running");
            while(true){
                synchronized(DeadLock.obj2){
                    System.out.println("Lock2 lock obj2");
                    Thread.sleep(3000);
                    synchronized(DeadLock.obj1){
                        System.out.println("Lock2 lock obj1");
                    }
                }
            }
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}
```

## 预防死锁
- 打破互斥条件
    - 即允许进程同时访问某些资源。但是，有的资源是不允许被同时访问的，像打印机等等，这是由资源本身的属性所决定的。所以，这种办法并无实用价值
- 打破不可抢占条件
    - 即允许进程强行从占有者那里夺取某些资源。就是说，当一个进程已占有了某些资源，它又申请新的资源，但不能立即被满足时，它必须释放所占有的全部资源，以后再重新申请。它所释放的资源可以分配给其它进程。这就相当于该进程占有的资源被隐蔽地强占了。这种预防死锁的方法实现起来困难，会降低系统性能
- 打破占有且申请条件
    - 可以实行资源预先分配策略。即进程在运行前一次性地向系统申请它所需要的全部资源。如果某个进程所需的全部资源得不到满足，则不分配任何资源，此进程暂不运行。只有当系统能够满足当前进程的全部资源需求时，才一次性地将所申请的资源全部分配给该进程。由于运行的进程已占有了它所需的全部资源，所以不会发生占有资源又申请资源的现象，因此不会发生死锁。但是，这种策略也有如下缺点：
        - 在许多情况下，一个进程在执行之前不可能知道它所需要的全部资源。这是由于进程在执行时是动态的，不可预测的；
        - 资源利用率低。无论所分资源何时用到，一个进程只有在占有所需的全部资源后才能执行。即使有些资源最后才被该进程用到一次，但该进程在生存期间却一直占有它们，造成长期占着不用的状况。这显然是一种极大的资源浪费；
        - 降低了进程的并发性。因为资源有限，又加上存在浪费，能分配到所需全部资源的进程个数就必然少了。
- 打破循环等待条件
    - 实行资源有序分配策略。采用这种策略，即把资源事先分类编号，按号分配，使进程在申请，占用资源时不会形成环路。所有进程对资源的请求必须严格按资源序号递增的顺序提出。进程占用了小号资源，才能申请大号资源，就不会产生环路，从而预防了死锁。这种策略与前面的策略相比，资源的利用率和系统吞吐量都有很大提高，但是也存在以下缺点：
        - 限制了进程对资源的请求，同时给系统中所有资源合理编号也是件困难事，并增加了系统开销；
        - 为了遵循按编号申请的次序，暂不使用的资源也需要提前申请，从而增加了进程对资源的占用时间。

# ThreadLocal
- ThreadLocal类中维护一个Map，用于存储每一个线程的变量副本，Map中元素的键为线程对象，而值为对应线程的变量副本
- 为每个线程维护一个本地变量。用于线程间的数据隔离，它为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。
```
public class Test {

    private static ThreadLocal<String> threadLocal;

    public static void main(String[] args) {

        threadLocal = new ThreadLocal<String>() {

            @Override
            protected String initialValue() {
                return "初始化值";
            }

        };
        
        for (int i = 0; i < 10; i++){
            new Thread(new MyRunnable(), "线程"+i).start();
        }

    }

    public static class MyRunnable implements Runnable {

        @Override
        public void run() {
            String name = Thread.currentThread().getName();
            System.out.println(name + "的threadLocal"+ ",设置为" + name);
            threadLocal.set(name);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {}
            System.out.println(name + ":" + threadLocal.get());
        }

    }

}
```
```
线程1的threadLocal,设置为线程1
线程4的threadLocal,设置为线程4
线程3的threadLocal,设置为线程3
线程2的threadLocal,设置为线程2
线程0的threadLocal,设置为线程0
线程6的threadLocal,设置为线程6
线程5的threadLocal,设置为线程5
线程7的threadLocal,设置为线程7
线程8的threadLocal,设置为线程8
线程9的threadLocal,设置为线程9
线程3:线程3
线程4:线程4
线程8:线程8
线程6:线程6
线程2:线程2
线程5:线程5
线程9:线程9
线程1:线程1
线程7:线程7
线程0:线程0
```

# synchronized
## synchronized锁什么
- 对于普通同步方法，锁是当前实例对象；
- 对于静态同步方法，锁是当前类的Class对象；
- 对于同步方法块，锁是括号中配置的对象；
- 当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。synchronized用的锁是存在Java对象头里的MarkWord，通常是32bit或者64bit，其中最后2bit表示锁标志位

## synchonized(this)和synchonized(object)区别
其实并没有很大的区别，synchonized(object)本身就包含synchonized(this)这种情况，使用的场景都是对一个代码块进行加锁，效率比直接在方法名上加synchonized高一些（下面分析），唯一的区别就是对象的不同
- 当两个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块
- 然而，当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的非synchronized(this)同步代码块。
- 尤其关键的是，当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对object中所有其它synchronized(this)同步代码块的访问将被阻塞
- 当一个线程访问object的一个synchronized(this)同步代码块时，它就获得了这个object的对象锁。结果，其它线程对该object对象所有同步代码部分的访问都被暂时阻塞。

## Synchronize作用于方法和静态方法区别
- 在static方法前加synchronized：静态方法属于类方法，它属于这个类，获取到的锁，是属于类的锁。
- 在普通方法前加synchronized：非static方法获取到的锁，是属于当前对象的锁。
- 类锁和对象锁不同，synchronized修饰不加static的方法，锁是加在单个对象上，不同的对象没有竞争关系；修饰加了static的方法，锁是加载类上，这个类所有的对象竞争一把锁。
```
private void test() {
    final TestSynchronized test1 = new TestSynchronized();
    final TestSynchronized test2 = new TestSynchronized();
    Thread t1 = new Thread(new Runnable() {

        @Override
        public void run() {
            test1.method01("a");
            //test1.method02("a");
        }
    });
    Thread t2 = new Thread(new Runnable() {

        @Override
        public void run() {
            test2.method01("b");
            //test2.method02("a");
        }
    });
    t1.start();
    t2.start();
}

private static class TestSynchronized{
    private int num1;
    public synchronized void method01(String arg) {
        try {
            if("a".equals(arg)){
                num1 = 100;
                System.out.println("tag a set number over");
                Thread.sleep(1000);
            }else{
                num1 = 200;
                System.out.println("tag b set number over");
            }
            System.out.println("tag = "+ arg + ";num ="+ num1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static int  num2;
    public static synchronized void method02(String arg) {
        try {
            if("a".equals(arg)){
                num2 = 100;
                System.out.println("tag a set number over");
                Thread.sleep(1000);
            }else{
                num2 = 200;
                System.out.println("tag b set number over");
            }
            System.out.println("tag = "+ arg + ";num ="+ num2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

//调用method01方法打印日志【普通方法】
tag a set number over
tag b set number over
tag = b;num =200
tag = a;num =100


//调用method02方法打印日志【static静态方法】
tag a set number over
tag = a;num =100
tag b set number over
tag = b;num =200
```

# volatile
- 轻量级锁。synchronized是阻塞式同步，在线程竞争激烈的情况下会升级为重量级锁。而volatile就可以说是java虚拟机提供的最轻量级的同步机制。
- 被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。
## 线程在工作内存进行操作后何时会写到主内存中
- Java内存模型告诉我们，各个线程会将共享变量从主内存中拷贝到工作内存，然后执行引擎会基于工作内存中的数据进行操作处理。
- 这个时机对普通变量是没有规定的，而针对volatile修饰的变量给java虚拟机特殊的约定，线程对volatile变量的修改会立刻被其他线程所感知，即不会出现数据脏读的现象，从而保证数据的“可见性”。 

## volatile的happens-before关系
volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读

## 一个int变量，用volatile修饰，多线程去操作++，线程安全吗
- 不安全,volatile只能保证可见性，并不能保证原子性.
- i++实际上会被分成多步完成：
    1. 获取i的值；
    2. 执行i+1；
    3. 将结果赋值给i
- volatile只能保证这3步不被重排序

# CAS
CAS即compare and swap的缩写，中文翻译成比较并交换。CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。自旋就是不断尝试CAS操作直到成功为止。
## CAS实现原子操作会出现什么问题
- ABA问题。因为CAS需要在操作之的时候，检查值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成B，有变成A，那么使用CAS进行检查时会发现它的值没有发生变化，但实际上发生了变化。ABA问题可以通过添加版本号来解决。Java 1.5开始，JDK的Atomic包里提供了一个类AtomicStampedReference来解决ABA问题。
- 循环时间长开销大。pause指令优化。
- 只能保证一个共享变量的原子操作。可以合并成一个对象进行CAS操作

# 多线程同步
假如有n个网络线程，需要当n个网络线程完成之后，再去做数据处理，你会怎么解决？
- 这种情况可以可以使用thread.join()；join方法会阻塞直到thread线程终止才返回。更复杂一点的情况也可以使用CountDownLatch，CountDownLatch的构造接收一个int参数作为计数器，每次调用countDown方法计数器减一。做数据处理的线程调用await方法阻塞直到计数器为0时。

# Runnable接口和Callable接口
Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已；Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果

# 乐观锁和悲观锁
- 乐观锁：就像它的名字一样，对于并发间操作产生的线程安全问题持乐观状态，乐观锁认为竞争不总是会发生，因此它不需要持有锁，将比较-替换这两个动作作为一个原子操作尝试去修改内存中的变量，如果失败则表示发生冲突，那么就应该有相应的重试逻辑
- 悲观锁：还是像它的名字一样，对于并发间操作产生的线程安全问题持悲观状态，悲观锁认为竞争总是会发生，因此每次对某资源进行操作时，都会持有一个独占的锁，就像synchronized，直接上了锁就操作资源。

# 在Java内存模型有哪些可以保证并发过程的原子性、可见性和有序性的措施？
- 原子性（Atomicity）：一个操作要么都执行要么都不执行。
    - 可直接保证的原子性变量操作有：read、load、assign、use、store和write，因此可认为基本数据类型的访问读写是具备原子性的
    - 若需要保证更大范围的原子性，可通过更高层次的字节码指令monitorenter和monitorexit来隐式地使用lock和unlock这两个操作，反映到Java代码中就是同步代码块synchronized关键字
- 可见性（Visibility）：当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。
    - 通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方式来实现。
    - 提供三个关键字保证可见性：volatile能保证新值能立即同步到主内存，且每次使用前立即从主内存刷新；synchronized对一个变量执行unlock操作之前可以先把此变量同步回主内存中；被final修饰的字段在构造器中一旦初始化完成且构造器没有把this的引用传递出去，就可以在其他线程中就能看见final字段的值。
- 有序性（Ordering）：程序代码按照指令顺序执行。
    - 如果在本线程内观察，所有的操作都是有序的，指“线程内表现为串行的语义”；如果在一个线程中观察另一个线程，所有的操作都是无序的，指“指令重排序”现象和“工作内存与主内存同步延迟”现象。
    - 提供两个关键字保证有序性：volatile 本身就包含了禁止指令重排序的语义；synchronized保证一个变量在同一个时刻只允许一条线程对其进行lock操作，使得持有同一个锁的两个同步块只能串行地进入。